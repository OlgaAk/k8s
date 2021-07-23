Learning Kubernates <br>
Microservices Project

Steps:

1. Preparation

   install minikube (inkl kubectl, virtualbox) <br>
   make docker commandline work with minikube's docker daemon <br>
   minikube docker-env <br>
   eval $(minikube -p minikube docker-env) - every command line <br>

2. Docker

   pull image of frontend microservice <br>
   docker image pull richardchesterwood/k8s-fleetman-webapp-angular:release0-5\n <br>
   run container <br>
   docker container run -p 8080:80 -d richardchesterwood/k8s-fleetman-webapp-angular:release0-5\n <br>
   to access in browser check minikubes ip <br>
   minikube ip <br>
   kubernates <br>
   pod - the most basic unit of deployment. usually one docker container - one pod. <br>

3. Pod

   create a yaml file for pod <br>
   kubectl apply -f first-pod.yaml <br>
   kubectl apply -f . // all files <br>
   kubectl get all <br>
   kubectl describe pod/webapp // status info <br>
   kubectl exec webapp -- ls // show contents of the folder <br>
   kubectl -it exec webapp sh // open shell inside of contasiner <br>
   wget http://localhost:80 // request local server inside container <br>
   cat index.html // read file inside shell <br>
   if need delete: kubectl delete po webapp

4. Service

   create a yaml file for service with selector for pod <br>
   kubectl apply -f webapp-service.yaml <br>
   add field labels in the pod's yaml metadata matching the services selector (any key/value pair) <br>
   apply changes <br>
   check browser http://192.168.99.100:30080 (minikube ip and port from yaml which should be greater than 30tsd) <br>
   kubectl get po --show-labels <br>
   kubectl get po --show-labels -l release=0 <br>

5. ReplicaSet

   If a pod dies for some reason, replicaset will bring up a new one (or how many replicas were indicated) <br>
   Edit pod: put existing description under "template" field. Change Kind to Replicaset, version to apps/v1, add replica quantity, selector and name. <br>
   kubectl delete pods --all <br>
   kubectl apply -f . <br>
   kubectl get all <br>
   ReplicaSet creates a pod (or several pods, depends on how many replicas are desired) with a made-up name. If one pod crashes, another one is running and a new one is being created<br>
   kubectl describe rs webapp <br>

6. Deployments

   Deployment is more advanced than ReplicaSet, it creates a new replicaset (name + -random id) which creates pods (name + -random id + -random id)<br>
   Deployment supports rollback (last 10 revisions), because it doesn't delete replicaset, just sets raplicas quantity to zero (if the new replica is healthy and in ready state). No downtime<br >
   kubectl delete rs webapp <br>
   In pods.yaml change the kind to Deployment<br>
   kubectl apply -f pods.yaml<br>
   Change the docker image in pod to new release and apply.<br>
   kubectl rollout status deployment webapp<br>
   kubectl rollout history deployment webapp<br>
   kubectl rollout undo deployment webapp --to-revision=2<br>
   But after rollback the yaml files don`t match the real situation! Use only in emergency<br>

7. Connecting containers, Service discovery (ex. backend+ DB)

   Backend App = Pod + Service<br>
   DB = Pod + Service<br>
   Kube DNS Service = key (pod name) &value (ip address) pairs<br>
   Namespaces are like packages<br>
   default namespace<br>
   kubectl get namespaces || ns<br>
   kubectl get all -n kube-system // shows kube dns service<br>

   Create pod and service(ClusterIP) for mysql, apply<br>
   kubectl exec -it pod/webapp-5f97db5744-gwg76 sh<br>
   cat /etc/resolv.conf // shows namespace ip<br>
   nslookup database // shows ip of the service named database<br>
   install msql client inside docker container:<br>
   apk update<br>
   apk add mysql-client<br>
   mysql -h database -uroot -ppassword fleetman<br>

   Fully qualified domain names: database. default.svc.cluster.local<br>

8. Microservices Architecture

   A system for a transport company: Vehicles report their current position every 10sec to a server.<br>
   Backend Part:<br>
   1)Position Simulator Microservice (simulates real vehicles, reads data from files in infinite loop)<br>
   2)Active MQ Microservice<br>
   3)Posiotion Traker Microservice reads positions from Acrive MQ, calculates speed, keeps history<br>
   Frontend<br>
   4)Angular App<br>
   "Middle"<br>
   5)API Gateway - single point of entry for frontend, delegates calls to backend

   kubectl delete -f .<br>
   deploy queue as Deployment with 1 replicas, port: 61616<br>
   deploy position-simulator as Deployment with 1 replicas, an env variable. no service needed because it only sends messages to queue<br>

   Logs

   kubectl logs pod-name<br>
   kubectl logs -f pod-name // follow logs<br>

   deploy position tracker as Deployment with a Service type ClusterIP. It starts consuming messages from ActiveMQ. Change type to NodePort to check REST API http://192.168.99.100:30020/vehicles/ // minukubeIp:nodePort/ <br>
   In the Spring app properties set are spring.activemq.broker-url=tcp://fleetman-queue:61616 // deploymentName:port<br>
   Deploy api gateway as Deployment with a Service type ClusterIP.<br>
   Deploy angular webapp as Deployment with a Service type NodePort. <br>
