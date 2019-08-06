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

######Setting up Load Balancer
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
    **kind: ReplicationController**
    metadata:
      name: helloworld-controller
    spec:
      **replicas: 2**
      selector:
        app: helloworld
      **template:**
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






