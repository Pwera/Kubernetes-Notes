# Kubernetes
Kubernetes is and open source orchestration system for containers.

It lets you schedule containers on a cluster of machines.

Kubernetes can run multiple contianers on one machine.

Kubernetes will manage the state of these containers.

Can start the contaienr on specific nodes.

Will restart container when it gets killed.

Can move containers from one node to another node.

Containers inside same pod can communicate with localhost.

Kubelets are responsible to launch the pods. It's going to connect to the master node to get this information.

Kube-proxy is going to feed its information about what pods are on nodes to iptables. Iptables is the firewall in Linux and it can also route traffic.
So whenever a new pod is launched the kube-proxy is going to change the Iptables rules to make sure that the pod is routable within the cluster.



## Kubernetes Setup

###### Minikube
Minikube is a tool that makes it easy to run Kubernetes locally.
Minikube runs a single-node Kubernetes cluster inside a Linux VM.
https://github.com/kubernetes/minikube
It cannot spin up a production cluster, it's a one node machine with no high availability.

``` 
     curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.3.0/minikube-linux-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
```

``` 
     minikube version
```
Checks if Minikube is properly installed

``` 
     minikube start --wait=false && \
     cat ~/.kube/config
```

``` 
     kubectl run hello-world --image=gcr.io/google_containers/echoserver:1.4 --port=8080
```
Hello World in Kubernetes.

``` 
     kubectl expose deployment hello-world --type=NodePort
```
Expose service to access externally. Deployment goes to non master nodes.

``` 
     minikube stop
```
Stop Minikube


###### Pod  
Pod describes an application running on Kubernetes. 
Can container on or more containers.
``` 
    apiVersion: v1
    kind: Pod
    metadata:
      name: nodehelloworld.example.com
      labels:
        app: helloworld
    spec:
      containers:
      - name: k8s-demo
        image: wardviaene/k8s-demo
        ports:
        - name: nodejs-port
          containerPort: 3000
```

``` 
     kubectl create -f <path_to_file>.yml
```
Create Pod on the Kubernetes clsuter.

``` 
     kubectl get pod
```
Get information about all running pods.

``` 
     kubectl describe pod <pod>
```
Describe one pod.

``` 
     kubectl expose pod <pod> --port=444 --name=frontend
```
Expose the port of a pod. Creates a new service.

``` 
     kubectl port-forward <pod> 8080
```
Forward port the exposed pod port to local machine.

``` 
     kubectl attach <podname> -i
```
Attach to the pod.

``` 
     kubectl exec <pod> --command
```
Wxecute a command on the pod

``` 
    kubectl label pods <pod> mylabel=awesome     
```
Add label to a pod.

``` 
     kubectl run -i -tty busybox --image=busybox --restart=Never --sh
```
Run a shell in a pod.

``` 
    apiVersion: v1
    kind: Service
    metadata:
      name: helloworld-service
    spec:
      ports:
      - port: 31001
        nodePort: 31001
        targetPort: nodejs-port
        protocol: TCP
      selector:
        app: helloworld
      type: NodePort
```
kubectl create -f app/helloworld.yml

``` 
     kubectl get pod
     kubectl describe pod nodehelloworld.example.com
     kubectl port-forward nodehelloworld.example.com 8081:3000
```

``` 
     kubectl expose pod nodehelloworld.example.com --type=NodePort --name <service_name>
```
NodePort means that we can directly access port 3000 on the Kubernetes and it will redirect to this pod.

``` 
     minikube service <service_name> --url
```
Retrive service.

``` 
     kubectl attach <service_name>
```
Attach to running process. Acces to logs.

``` 
     kubectl exec <service_name> <command>
     kubectl exec nodehelloworld --ls /app
```
Runn a command inside running service.

###### Setting up Load Balancer
Load Balancer will route the traffic to the correct pod on Kubernetes.
``` 
     kubectl create -f k8s/elb/helloworld.yml
     kubectl create -f k8s/elb/helloworld-service.yml
```
The LoadBalancer Service will create a port that is open on every node, that can be used by the AWS LoadBalancer to connect. That port is owned by the Service, so that the traffic can be routed to the correct pod


###### Scaling pods
Scaling in Kubernetes can be done using the Replication Controller.
Replication Controller will ensure a specific of pod replicas will run at all time.
A pods created with the replica controller will automatically be replaced if they fail, get delected, or are terminated.
We can horizontally scalling only stateless pods.

```
    apiVersion: v1
    kind: ReplicationController
    metadata:
      name: helloworld-controller
    spec:
      replicas: 2
      selector:
        app: helloworld
      template:
        metadata:
          labels:
            app: helloworld
        spec:
          containers:
          - name: k8s-demo
            image: wardviaene/k8s-demo
            ports:
            - name: nodejs-port
              containerPort: 3000

    kubectl create -f k8s/replication-controller/helloworld-repl-controller.yml && \
    kubectl get pods
    kubectl delete pod <name>
```

``` 
     kubectl scale --replicas=4 -f k8s/replication-controller/helloworld-repl-controller.yml && \
     kubectl get pods
```

###### Deployments
Replication Set is the next generation of Replication Controller.
It support a new selector that can selection based on filtering according a set values, e.g environment.
Replica Set i used by the Deployment object.
When using the deployment object, you define the state of application. Kubernetes will then make sure the cluster matches your desired state.
With deployment object you can:
- Create a deployment
- Update a deployment
- Do rolling updates (zero downtime deplyments)
- Rollback
- Pause/ Resume
``` 
     apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: helloworld-deployment
    spec:
      replicas: 3
      template:
        metadata:
          labels:
            app: helloworld
        spec:
          containers:
          - name: k8s-demo
            image: wardviaene/k8s-demo
            ports:
            - name: nodejs-port
              containerPort: 3000

```


``` 
     kubectl get deployments 
```
Get information on current deployments.

``` 
     kubectl get rs
```
Get information about the replisa sets.

``` 
     kubectl get pods --show-labels
```
Get pods and show labels attached to those pods.

``` 
     kubectl rollout status deployment/helloworld-deployment
```
Get deployment status.


``` 
     kubectl create -f k8s/deployment/helloworld.yml
     kubectl get deployments
     kubectl get rs
     kubectl get pods
     kubectl expose deployment helloworld-deployment --type=NodePort
     kubectl get service
     kubectl describe service helloworld-deployment
     minikube service helloworld-deployment --url
     kubectl set image k8s/deployment/helloworld-deployment k8s-demo=wardviaene/k8s-demo:2
     kubectl rollout status k8s/deployment/helloworld-deployment
     kubectl history k8s/deployment/helloworld-deployment
     kubectl undo k8s/deployment/helloworld-deployment
```
Example of deployment

###### Services

Pods are very dynamic, they come and go  on the Kubernetes cluster.
When using a Replication Controller, pods are terminated and created during scalling operations.
When using Deployments, when updating the image verson, pods are terminated and new pods take the place of older pods.
Pods should neber be accessed directly, but always through a Service.
A service is th logical bridge between the "mortal" pods and other service or end-user.
When using "kubectl expose" command, you created a new service for Pods, so it could be accessed externally.
Creating a service will create an endpoint for pods:
- a CluserIP: a virtual IP address only reachable from within the cluster (default)
- a NodePort: a port that is the same on each node ais also reachable externally
- a LoadBalancer will route external trafiic to every node on the NodePort.

``` 
     kubectl create -f k8s/first-app/helloworld.yml
     kubectl get pods
     kubectl describe pod nodehelloworld.example.com 
     kubectl create -f k8s/first-app/helloworld-nodeport-service.yml
     minikube service helloworld-service --url
     kubectl describe svc helloworld-service
     kubectl delete svc helloworld-service
```
Example of Service, using static node port

###### Labels
Labels are key/value pairs that can be attached to objects.
Labels are like tags in AWS or other cloud providers, used to tag resources.
Label can be used to tag nodes, once nodes are tagged, you can use label selectors to let pods only run on specific nodes.

First step, add a label or multiple labels to nodes.
Secondly all pod that uses those labels.

``` 
    kubectl get nodes --show-labels
    kubectl create -f k8s/deployment/helloworld-nodeselecotr.yml
    kubectl get deployments
    kubectl get pods
    kubectl label nodes minikube hardware=high-spec
    kubectl get nodes --show-labels

```
Example of lables

###### Health checks
There are two different type of health check:
- Running a command in the container periodically
- Periodic check on a URL

``` 
    kubectl create -f k8s/deployment/helloworld-healthcheck.yml
    kubectl get deployments
    kubectl get pods
    kubectl edit deployment/helloworld-deployment

```
Example of Health checks

###### Readiness Probe.
Besides livenessProbes you can also use readinessProbe on a container within a Pod.
LivenessProbes indicate wheter container is running.
If the check fail, the container will be restarted.
ReadinessProbe indicateswhether the container is ready to serve requests.
If the check fails the container will not be restarted, but the Pod's IP address will be removed ffom the service, so it'll not serve any requesty anymore.
``` 
     kubectl create -f k8s/deployment/helloworld-liveness-readiness.yml && watch -n1 kubectl get pods
```

###### Pod state
State Running
- Pod has been bound to a node.
- All containers have been created.
- At least on container is still running, or is starting/restarting.
State Pending:
- Pod has been accepted but is not running
- Happens when the container image is still downloading
- If the pod cannot be scheduled because of resource constraints (leak of cpu/ memmory resources)
State Succeeded
- All containers within this pod have been terminated successfully and will not be restarted
State Failed
- All containers within this pod have been terminated, and at least oe container returned a failure code
State Unknown
- The state of the pod couldn't be detemined
- Eg. network problems
``` 
     kubectl pod <pod_name> -n kube-system
```
Get pod conditions.
There are 5 different types of PodConditions:
- PodScheduled
- Ready
- Initialized
- Unschedulable
- ContainersReady
There also container state
``` 
     kubectl get pod <pod_name> -n kube-system -o yaml
```
There are 3 different types of containers state:
- Running
- Terminated
- Waiting

######  Secrets
Secrets provides a way in Kubernetes to distribute credentials, keys, passwords, etc.
Kubernetes itself uses this Secrets mechanism to provide the credentials to access the internal API.
We can use as environmental variables, as file in pod. Can be used for dotenv files.
``` 
     echo -n "root > ./username
     echo -n "password" > ./password
     kubectl create secret generic db-user-pass --from-file=./username --from-file=./password
```

A secret can also be an SSH key or an SSL certificate.

``` 
     kubectl create -f k8s/deployment/helloworld-secrets.yml
     kubectl create -f k8s/deployment/helloworld-secrets-volumes.yml
     kubectl get pods
     kubectl exec <pod_name> -i -t -- /bin/bash
     cat /etc/creds/username

```
Example of secrets

``` 
     kubectl create -f k8s/wordpress/wordpress-secrets.yml
     kubectl create -f k8s/wordpress/wordpress-single-deployment-no-volumes.yml
     kubectl create -f k8s/wordpress/wordpress-service.yml
     minikube service wordpress-service --url
```
Wordpress  example with no volumes


``` 
     minikube dashboard --url
```
Show Kubernetes dashboard

######  Service Discovery using DNS.
As on Kubernetes 1.3, DNS is built-in service launched automatically using the addon manager.
The addons are on master node.
The DNS service can be used within pods to find other services running on the same cluster.
Multiple container within 1 pod don't need the service, as they can contact each other directly.
A container in the same pod can connect the port of the other container directly using localhost:port.
To make DNS work, a pod will need a Service Definition.
Default stands for the deafult namespace Pods and services can be launched in different namespaces - to logically separate cluster.

``` 
     kubectl create -f k8s/service-discovery/secrets.yml
     kubectl create -f k8s/service-discovery/database.yml
     kubectl create -f k8s/service-discovery/database-service.yml
     kubectl create -f k8s/service-discovery/helloworld-db.yml
     kubectl create -f k8s/service-discovery/helloworld-db-service.yml
     minikube service helloworld-db-service --url
     kubectl logs <deploymen_name>
```
Example: Service Discovery

``` 
     .
```


###### Kops (Kubernetes Operations)
Tool used to spin up a highly available production cluster.
Used to setup Kubernetes on AWS.
Works only on Mac / Linux.

``` 
     .
```

``` 
     .
```

``` 
     .
```
