Kubernates Microservices Project

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
   kubectl apply -f . #all files <br>
   kubectl get all <br>
   kubectl describe pod/webapp // status info <br>
   kubectl exec webapp -- ls // show contents of the folder <br>
   kubectl -it exec webapp sh // open shell inside of contasiner <br>
   wget http://localhost:80 // request local server inside container <br>
   cat index.html // read file inside shell <br>

4. Service

   create a yaml file for service with selector for pod <br>
   kubectl apply -f webapp-service.yaml <br>
   add field labels in the pod's yaml metadata matching the services selector (any key/value pair) <br>
   apply changes <br>
   check browser http://192.168.99.100:30080 (minikube ip and port from yaml which should be greater than 30tsd) <br>
   kubectl get po --show-labels <br>
   kubectl get po --show-labels -l release=0 <br>
