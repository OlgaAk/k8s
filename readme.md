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

9. Persistance

   Add mongo stack (deployment + service ClusteIP) with official mongo docker image<br>
   Update release tags.<br>

   ![screenshot](./screenshot.jpg)

   In order to persist data even if mongo pod crashes we need to copy data to a folder on te host machine.<br>
   Add volumeMounts to the mongo container's spec, with a mountPath: /data/db<br>
   Add volumes field after containers with hostPath: with type: DirectoryOrCreate and some path in virtual box (ex /mnt)<br>
   To check if files were copied open VirtualBox, double click to open terminal, login under docker tcuser, go cd / than ls<br>
   Move volume config to separate yaml with persistentVolumeClaim + persistentVolume, which are binded by the same storageClassName (accessmode should be the same, storage same or bigger in pv)<br>
   kubectl get pv<br>
   kubectl get pvc<br>

   10. From minikube to AWS

   Create account<br>
   Node is a fisical server (called ec2 instance)<br>
   Master node is responsible for scheduling nodes<br>
   Ebs - persistent volume<br>

   Kops

   Kops - Kubernates Operations sets up a production k8s cluster<br>
   Insead of running kops on local machine set Linux instance in AWS<br>
   In AWS launch instance - Amazon Linux 2 AMI (the smallest), add tag (name-bootstrap), in security select source my ip, create key pair and download. View instances, copy the public api. Open your computer terminal, copy the key pair into the folder with k8s project.<br>
   To login run: ssh -i keypair.pem ec2-user@<copied ip address here><br>
   (may be needed: chmod go-rwx keypair.pem)<br>
   https://kops.sigs.k8s.io/getting_started/install/<br>
   curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64<br>
   chmod +x kops<br>
   sudo mv kops /usr/local/bin/kops<br>

   Kubectl<br>
   https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-kubectl-binary-with-curl-on-linux<br>
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"<br>
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl<br>
   kubectl version --client<br>

   Setup IAM User<br>
   In command line:<br>
   aws iam create-group --group-name kops + other commands<br>
   https://kops.sigs.k8s.io/getting_started/aws/<br>
   Or in UI: services menu -> IAM (under Security) - Groups - Create new Group "kops" - select needed policies - create group. Under Users - Add User "kops", access type "programmatic access" - Permissions add user to Group "kops". Access key id, Secret access key.<br>
   Run: aws configure // enter keys here<br>
   Select region (in UI go to Services -EC2 - in upper right corner select region, search in docs for the correct corresponding name like eu-west-2)<br>
   aws iam list-users<br>
   export AWS_ACCESS_KEY_ID=$(aws configure get aws_access_key_id)<br>
   export AWS_SECRET_ACCESS_KEY=$(aws configure get aws_secret_access_key)<br>
   Skip DNS section (not needed in recent versions)<br>

   State storage<br>
   Setup S3 Bucket to store data in kops<br>
   In UI - Services - S3 - Create bucket (with a globally unique name like yourname-state-storage)<br>

   Create cluster<br>
   Env variables:<br>
   export NAME=fleetman.k8s.local //use k8s.local!<br>
   export KOPS_STATE_STORE=s3://yourname-state-store<br>
   Cluster configuration: <br>
   Check your availability zones (there are 2-4 datacenters in each region): aws ec2 describe-availability-zones --region us-west-1<br>
   Add all zones to config comma separated:<br>
   kops create cluster --zones [zones here] ${NAME}<br>
