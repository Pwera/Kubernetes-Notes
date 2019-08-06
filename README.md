# Kubernetes
Kubernetes is and open source orchestration system for containers.
It lets you schedule containers on a cluster of machines.
Kubernetes can run multiple contianers on one machine.
Kubernetes will manage the state of these containers.
Can start the contaienr on specific nodes.
Will restart container when it gets killed.
Can move containers from one node to another node.


# Kubernetes Setup

# Minikube
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


# Pod  
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

Setting up Load Balancer
Load Balancer will route the traffic to the correct pod on Kubernetes.
``` 
     .
```

``` 
     .
```



# Kops (Kubernetes Operations)
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






