Kubernates Microservices Project

Steps:

1. Preparation
   install minikube (inkl kubectl, virtualbox)
   make docker commandline work with minikube's docker daemon
   minikube docker-env
   eval $(minikube -p minikube docker-env) - every command line

2. Docker
   pull image of frontend microservice
   docker image pull richardchesterwood/k8s-fleetman-webapp-angular:release0-5\n
   run container
   docker container run -p 8080:80 -d richardchesterwood/k8s-fleetman-webapp-angular:release0-5\n
   to access in browser check minikubes ip
   minikube ip
   kubernates
   pod - the most basic unit of deployment. usually one docker container - one pod.

3. Pod
   create a yaml file for pod
   kubectl apply -f first-pod.yaml
   kubectl get all
   kubectl describe pod/webapp // status info
   kubectl exec webapp -- ls // show contents of the folder
   kubectl -it exec webapp sh // open shell inside of contasiner
   wget http://localhost:80 // request local server inside container
   cat index.html // read file inside shell

4. Service
   create a yaml file for service with selector for pod
   kubectl apply -f webapp-service.yaml
   add field labels in the pod's yaml metadata matching the services selector (any key/value pair)
   apply changes
   check browser http://192.168.99.100:30080 (minikube ip and port from yaml which should be greater than 30tsd)
