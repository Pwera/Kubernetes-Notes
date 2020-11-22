# Kubernetes
<i>Kubernetes</i> is and open source orchestration system for containers. It lets you schedule containers on a cluster of machines. Kubernetes can run multiple contianers on one machine. Kubernetes will manage the state of these containers. Can start the contaienr on specific nodes. Will restart container when it gets killed. Can move containers from one node to another node. Containers inside same pod can communicate with localhost.

<i>Kubelets</i> are responsible to launch the pods. It's going to connect to the master node to get this information.

<i>Kube-proxy</i> is going to feed its information about what pods are on nodes to iptables. Iptables is the firewall in Linux and it can also route traffic.
So whenever a new pod is launched the kube-proxy is going to change the Iptables rules to make sure that the pod is routable within the cluster.



## Kubernetes Setup

###### k3s
// TODO

###### Minikube
<i>Minikube</i> is a tool that makes it easy to run Kubernetes locally.
<i>Minikube</i> runs a single-node Kubernetes cluster inside a Linux VM.
https://github.com/kubernetes/minikube
It cannot spin up a production cluster, it's a one node machine with no high availability.

<table>
<tr>
<td>Minikube version
<td>

```bash
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v1.3.0/minikube-linux-amd64 && 
chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
```
</tr>
<tr>
<td>Checks if Minikube is properly installed
<td>

``` bash
minikube start --wait=false && \
cat ~/.kube/config 
# --driver=none if already running on VM
# preferred driver: docker
```
<td>

```bash
Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, dashboard, default-storageclass
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
```
<tr>
<td>Hello World in Kubernetes.
<td>

```bash
kubectl create deployment hello-world --image=gcr.io/google_containers/echoserver:1.4 --port=8080
```
<td>

```bash
pod/hello-world created
```
<tr>
<td>Expose service to access externally. Deployment goes to non master nodes.
<td>

```bash
kubectl expose deployment hello-world --type=NodePort
```
<td>

```bash
service/hello-world exposed
```
<tr>
<td>Status
<td>

```bash
kubectl get all
```
<td>

``` yaml
NAME              READY   STATUS    RESTARTS   AGE
pod/hello-world   1/1     Running   0          62s

NAME                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
service/hello-world   ClusterIP   10.106.X.Y  <none>        8080/TCP   4s
service/kubernetes    ClusterIP   10.96.X.Y        <none>        443/TCP    4d2h
```
<tr>
<td>Show Kubernetes dashboard
<td>

```bash
minikube dashboard --url
```


<tr>
<td>Stop Minikube
<td>

```bash
minikube stop
```
</table>


###### Pod
Pod describes an application running on Kubernetes. 
Can contain one or more containers. Each pod is given an unique IP address. Containers in a pod can spean to each other on localhost. They share the same network, storage, process space.

<table>
<tr>
<td>Pod resource definition
<td>

```yaml
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
<tr>
<td>Create Pod on the Kubernetes cluster.
<td>

```bash
kubectl apply -f helloworld.yml
```
<td>

```
pod/nodehelloworld.example.com created
```
<tr>
<td>Get information about all running pods.
<td>

```bash
kubectl get pod
```
<td>

```bash
NAME                         READY   STATUS    RESTARTS   AGE
nodehelloworld.example.com   1/1     Running   0          52s
```
<tr>
<td>Describe one pod.
<td>

```bash
kubectl describe pod pod/nodehelloworld.example.com
```
<td>

```yaml
Name:         nodehelloworld.example.com
Namespace:    default
Priority:     0
Node:         minikube/192.168.Y.X
Start Time:   Thu, 19 Nov 2020 21:29:28 +0100
Labels:       app=helloworld
Annotations:  <none>
Status:       Running
IP:           172.17.Y.X
IPs:
  IP:  172.17.Y.X
Containers:
  k8s-demo:
    Container ID:   docker://2e92c3a27d78df3a59706ad5c89ce72469f6050eab7b6d004ab1ebf493182f99
    Image:          wardviaene/k8s-demo
    Image ID:       docker-pullable://wardviaene/k8s-demo@sha256:2c050f462f5d0b3a6430e7869bcdfe6ac48a447a89da79a56d0ef61460c7ab9e
    Port:           3000/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 19 Nov 2020 21:29:31 +0100
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8thtz (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             True
  ContainersReady   True
  PodScheduled      True
Volumes:
  default-token-8thtz:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8thtz
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  98s   default-scheduler  Successfully assigned default/nodehelloworld.example.com to minikube
  Normal  Pulling    98s   kubelet            Pulling image "wardviaene/k8s-demo"
  Normal  Pulled     96s   kubelet            Successfully pulled image "wardviaene/k8s-demo" in 2.1721431s
  Normal  Created    96s   kubelet            Created container k8s-demo
  Normal  Started    96s   kubelet            Started container k8s-demo
```
<tr>
<td>Expose the port of a pod. Creates a new service. By default type is <i>CLUSTER-IP</i>.
<td>

```bash
kubectl expose pod/nodehelloworld.example.com --port=444 --name=frontend
```
<tr>
<td>Forward port the exposed pod port to local machine.
<td>

```bash
kubectl port-forward pod/nodehelloworld.example.com 8080
```
<tr>
<td>Attach to the pod.
<td>

```bash
kubectl attach pod/nodehelloworld.example.com -i
```
<tr>
<td>Execute a command on the pod
<td>

```bash
kubectl exec --stdin --tty pod/nodehelloworld.example.com -- /bin/bash
```
<tr>
<td>Run a shell in a pod.
<td>

```bash
kubectl run -i -tty busybox --image=busybox --restart=Never --sh
```
</table>

###### Service
Kubernetes Service group Pods with shared IP, allow external access to Pods.

<table>
<tr>
<td>Service resource definition
<td>

```yaml
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
<tr>
<td>Create Pod Resource
<td>

```bash
kubectl apply -f first-app/helloworld.yml
```
<tr>
<td>Create Service Resource of type <i>NodePort</i>
<td>

```bash
kubectl apply -f=first-app/helloworld-nodeport-service.yml
```
<tr>
<td>Forward traffic to cluster for <i>NodePort</i> service.NodePort means that we can directly access port 3000 on the Kubernetes and it will redirect to this pod.
<td>

```bash
kubectl port-forward pod/nodehelloworld.example.com 8081:3000
```
<td>

```bash
Forwarding from 127.0.0.1:8081 -> 3000
Forwarding from [::1]:8081 -> 3000
```
<tr>
<td>Create Service Resource of type <i>LoadBalancer</i>.
The LoadBalancer Service will create a port that is open on every node, that can be used by the AWS LoadBalancer to connect. That port is owned by the Service, so that the traffic can be routed to the correct pod.
<td>

```bash
kubectl apply -f=first-app/helloworld-service.yml
```
<tr>
<td>Access load balancer

<td>

```bash
minikube service helloworld-service --url
```
</table>

###### Scaling pods
Scaling in Kubernetes can be done using the <i>Replication Controller</i>.
<i>Replication Controller</i> will ensure a specific of pod replicas will run at all time.
A pods created with the replica controller will automatically be replaced if they fail, get delected, or are terminated.
We can horizontally scalling only stateless pods.

<table>
<tr>
<td> ReplicationController <br>resource definition
<td>

```yaml
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
```
<tr>
<td>Create <i>ReplicationController</i>
<td>

```bash
kubectl create -f k8s/replication-controller/helloworld-repl-controller.yml
```
<tr>
<td>Status
<td>

```bash
kubectl get all
```
<td>

```yaml
NAME                              READY   STATUS    RESTARTS   AGE
pod/helloworld-controller-blpf9   1/1     Running   0          6m50s
pod/nodehelloworld.example.com    1/1     Running   0          32m

NAME                                          DESIRED   CURRENT   READY   AGE
replicationcontroller/helloworld-controller   2         2         2       6m51s

NAME                         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/helloworld-service   LoadBalancer   10.107.X.Y   <pending>     80:32155/TCP   32m
service/kubernetes           ClusterIP      10.96.X.Y       <none>        443/TCP        4d6h
```
<tr>
<td>Scale in imperative way
<td>

```bash
kubectl scale --replicas=4 -f k8s/replication-controller/helloworld-repl-controller.yml && \
kubectl get pods
```
</table>


###### Deployments

![Kubernetes Glossary](/k8s/imgs/kubernetesGlossary.png)

<i>Replication Set</i> is the next generation of Replication Controller.
It support a new selector that can selection based on filtering according a set values, e.g environment.
<i>Replica Set</i> is used by the <i>Deployment</i> object.
When using the deployment object, you define the state of application. Kubernetes will then make sure the cluster matches your desired state.
With deployment object you can:
- Create a deployment
- Update a deployment
- Do rolling updates (zero downtime deplyments)
- Rollback
- Pause/ Resume
Deployment creates Replica Set, Replica Set creates Pods.
One key feature that Deployments provide that ReplicaSets do not provide is rolling updates.

A ReplicaSet can only define a single Pod template; if you want to roll out a new version
of your application, you need to create a new Deployment that defines this Pod template.

Imperative commands enable users to quickly create, update, and delete Kubernetes objects.
Imperative commands are easiest to learn and therefore present the lowest barrier to entry.
For example, to create a Pod that runs a specific container—in this case, "nginx"–, you
can simply run the “kubectl run nginx --image nginx” command.
This command is simple to write and understand because it only contains a name and the image
that should be run as a container.
However, it doesn’t provide an audit trail, which is important in clusters so that operators
can know what changes are made.
It's also not very flexible: the options are limited, and additional commands must be run
to expose the Deployment as an externally available service.

Kubernetes deployment object is suitable for stateless application.
Deployments are more rebust and provide additional objects.
However, a minimalist Deployment can look exactly like a ReplicaSet.

<table>
<tr>
<td>

```bash
kubectl create deploy  nginx --image=nginx:1.16-alpine
```
<tr>
<td>

```bash
kubectl delete deploy/nginx
```
<tr>
<td>

```bash
kubectl set image deploy/nginx nginx=nginx:1.17-alpine
```
</table>

```yaml
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

<table>
<tr>
<td>Get information on current deployments.
<td>

```bash
kubectl get deployments 
```
<tr>
<td>Get information about the replisa sets.
<td>

```bash
kubectl get rs
```
<tr>
<td>Get pods and show labels attached to those pods.
<td>

```bash
kubectl get pods --show-labels
```

<tr>
<td>Get deployment status.
<td>

```bash
kubectl rollout status deployment/helloworld-deployment
```
</table>

```bash
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

Defines a DNS entry that can be used to refer to a group of pods.
Pods are very dynamic, they come and go  on the Kubernetes cluster.
When using a Replication Controller, pods are terminated and created during scalling operations.
When using Deployments, when updating the image verson, pods are terminated and new pods take the place of older pods.
Pods should neber be accessed directly, but always through a Service.
A service is th logical bridge between the "mortal" pods and other service or end-user.
When using "kubectl expose" command, you created a new service for Pods, so it could be accessed externally.
Creating a service will create an endpoint(!) for pods:
- a CluserIP: a virtual IP address only reachable from within the cluster (default)
- a NodePort: a port that is the same on each node ais also reachable externally
- a LoadBalancer will route external trafiic to every node on the NodePort.

```bash
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
<i>Labels</i> are key/value pairs that can be attached to objects.
Labels are like tags in AWS or other cloud providers, used to tag resources.
Label can be used to tag nodes, once nodes are tagged, you can use label selectors to let pods only run on specific nodes.

First step, add a label or multiple labels to nodes.
Secondly all pod that uses those labels.

```bash
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

<table>
<tr>
<td>Deployment with healthcheck (livenessProbe)
<td>

```yaml
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
        livenessProbe:
          httpGet:
            path: /
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30

```
<tr>
<td>Example of Health checks
<td>

```bash
kubectl create -f k8s/deployment/helloworld-healthcheck.yml
kubectl get deployments
kubectl get pods
kubectl edit deployment/helloworld-deployment
```
</table>

###### Readiness Probe.
Besides <i>livenessProbes</i> you can also use <i>readinessProbe</i> on a container within a Pod.
<i>LivenessProbes</i> indicate wheter container is running.
If the check fail, the container will be restarted.
<i>ReadinessProbe</i> indicateswhether the container is ready to serve requests.
If the check fails the container will not be restarted, but the Pod's IP address will be removed ffom the service, so it'll not serve any requesty anymore.

<table>
<tr>
<td>Deployment with<br> 
<i>readinessProbe</i>
<td>

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: helloworld-readiness
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
        livenessProbe:
          httpGet:
            path: /
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30
        readinessProbe:
          httpGet:
            path: /
            port: nodejs-port
          initialDelaySeconds: 15
          timeoutSeconds: 30

```
<tr>
<td>
<td>

```bash
kubectl create -f k8s/deployment/helloworld-liveness-readiness.yml
watch -n1 kubectl get pods
```
</table>


</table>


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
```bash
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
```bash
kubectl get pod <pod_name> -n kube-system -o yaml
```
There are 3 different types of containers state:
- Running
- Terminated
- Waiting

######  Secrets
Secrets provides a way in Kubernetes to distribute credentials, keys, passwords, etc.
Kubernetes itself uses this Secrets mechanism to provide the credentials to access the internal API.
We can use as environmental variables, as file in pod. Can be used for dotenv files. A secret can also be an SSH key or an SSL certificate.

<table>
<tr>
<td>Create <i>Secret</i>
<td>

```bash
echo -n "root > ./username
echo -n "password" > ./password
kubectl create secret generic db-user-pass --from-file=./username --from-file=./password
```
<tr>
<td>
<td>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
type: Opaque
data:
  username: cm9vdA==
  password: cGFzc3dvcmQ=

```
<tr>
<td><i>Deployment</i> with <i>Secret</i>
<td>

```yaml
apiVersion: v1
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
        volumeMounts:
        - name: cred-volume
          mountPath: /etc/creds
          readOnly: true
      volumes:
      - name: cred-volume
        secret: 
          secretName: db-secrets
```
<tr>
<td>
<td>

```bash
kubectl create -f k8s/deployment/helloworld-secrets.yml
kubectl create -f k8s/deployment/helloworld-secrets-volumes.yml
kubectl get pods
kubectl exec <pod_name> -i -t -- /bin/bash
cat /etc/creds/username
```
</table>

######  Service Discovery using DNS.
As on Kubernetes 1.3, DNS is built-in service launched automatically using the addon manager.
The addons are on master node.
The DNS service can be used within pods to find other services running on the same cluster.
Multiple container within 1 pod don't need the service, as they can contact each other directly.
A container in the same pod can connect the port of the other container directly using localhost:port.
To make DNS work, a pod will need a Service Definition.
Default stands for the deafult namespace Pods and services can be launched in different namespaces - to logically separate cluster.

```bash
kubectl create -f k8s/service-discovery/secrets.yml
kubectl create -f k8s/service-discovery/database.yml
kubectl create -f k8s/service-discovery/database-service.yml
kubectl create -f k8s/service-discovery/helloworld-db.yml
kubectl create -f k8s/service-discovery/helloworld-db-service.yml
minikube service helloworld-db-service --url
kubectl logs <deploymen_name>
```
Example: Service Discovery

######  ConfigMap
Configuration parameters that are not secret, can be put in a ConfigMap.
The ConfigMap key-value pairs can then be read by the app using:
- Environmental variables
- Container commandline arguments in the Pod configuration
- Using volumes
ConfigMap can also contain full configuration files.
Using ConfigMap:
- You can create a pod that exposes the configMap using a volume
- You can create a pod that exposes the configMap as environment variables.

<table>
<tr>
<td>reverseproxy.conf
<td>

```nginx
server {
    listen       80;
    server_name  localhost;

    location / {
        proxy_bind 127.0.0.1;
        proxy_pass http://127.0.0.1:3000;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}

```
<tr>
<td>Create configMap
<td>

``` bash
kubectl create configmap nginx-config --fromfile=k8s/configmap/reverseproxy.conf
kubectl get configmap nginx-config -o yaml
```
<tr>
<td><i>Pod</i> with <i>ConfigMap</i>
<td>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: helloworld-nginx
  labels:
    app: helloworld-nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.11
    ports:
    - containerPort: 80
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  - name: k8s-demo
    image: wardviaene/k8s-demo
    ports:
    - containerPort: 3000
  volumes:
    - name: config-volume
      configMap:
        name: nginx-config
        items:
        - key: reverseproxy.conf
          path: reverseproxy.conf
```
<tr>
<td>
<td>

```bash
kubectl create -f k8s/configmap/nginx.yml
kubectl create -f k8s/configmap/nginx-service.yml
```
</table>


######  Ingress

Route traffic to internal  services based on host and path.
Ingress is a solution available since Kubernetes 1.1 that allows inbound connections to the cluster. 
It's alternative to the external LoadBalancer and nodePorts
Ingress allows to easily expose services that need to be accessible form outsidde to the cluster.
With ingress you can run own ingress controller (basically a loadbalancer) within the Kubernetes cluster.
You can create ingress rules using the ingress object.

<table>
<tr>
<td><i>Ingress</i> Resource
<td>

```yaml
# An Ingress with 2 hosts and 3 endpoints
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: helloworld-rules
spec:
  rules:
  - host: helloworld-v1.example.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: helloworld-v1
            port:
              number: 80
  - host: helloworld-v2.example.com
    http:
      paths:
      - pathType: Prefix
        path: /
        backend:
          service:
            name: helloworld-v2
            port:
              number: 80

```
<tr>
<td>
<td>

```bash
kubectl create -f k8s/ingress/ingress.yml 
kubectl create -f k8s/ingress/nginx-ingress-controller.yml
kubectl create -f k8s/ingress/echoservice.yml 
kubectl create -f k8s/ingress/helloworld-v1.yml 
kubectl create -f k8s/ingress/helloworld-v2.yml 
kubectl get pod
```
</table>


###### Egress Gateway
Every traffic that needs to exit the service mesh needs to go via Egress Gateway.


###### External DNS
On public cloud providers, you can use the ingress controller to reduce the cost of LoadBalancer.
You can use one LB that capures all the external traffic and send it to the ingress controller.
The ingress controller can be configured to route thedifferent traffic to your apps based on http rules(host and prefixes).
This only works for https- based applications.

###### Volumes
Volumes in Kubernetes allow to store data outside the container.
When a container stops, all data on the container itself is lost.
Persistent volumes in Kubernetes allow to attach a volume to a container that will exists even when the container stops. Volumes than can re reattached.
To use volumes, you need to create a pod with a volume definition.

<table>
<tr>
<td>
Define<br>PersistentVolumeClaim
</td>
<td>

``` yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-pvc
spec:
  volumeName: host-pv
  accessModes:
    - ReadWriteOnce
  storageClassName: standard  
  resources:
    requests:
      storage: 1Gi 
 
```
</td>
</tr>
<tr>
<td>
Define<br>PersistentVolume
</td>
<td>

```yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  storageClassName: standard
  accessModes:
    - ReadWriteOnce  
  hostPath:
    path: /data/
```
</td>
</tr>
<tr>
<td>
Define

Deployment
with PersistentVolumeClaim
</td>
<td>

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloworld-deployment
  labels:
    app: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - name: k8s-demo
        image: ubuntu
        volumeMounts:
          - mountPath: /home
            name: demo-volume   
      volumes:
        - name: demo-volume
          persistentVolumeClaim: 
            claimName: host-pvc 
```
</td>
</tr>
</table>



###### Volumes AutoProvisioning
The Kubernetes plugins have the capability to provision storage.
This is done using the StorageClass object.

###### Pod Presets
Can inject information into pods at runtime.
Pod Presets are used to inject Kubernetes resources like Secrets, ConfigMaps, Volumes and environmental varibles.
When injecting environment variables and volumemounts, the pod presets will apply the changes to all containers within the pod.

```bash
kubectl create -f k8s/pod-presets/pod-presets.yml
kubectl get podpresets
kubectl create -f deployments.yml
```
Example of  Presets

###### StatefulSets
It's introduced to be able to run stateful applications.
A StatefulSet will allow stateful app to use DNS to find other peers.
```bash
kubectl create -f k8s/statefulset/cassandra.yaml
kubectl get pod
kubectl get pv
watch kubectl get pod
```


###### DeamonSets
Deamon Sets ensure that every single node in the Kubernetes cluster runs that same pod resource.
This is useful if you want to ensure that a certain pod is running on every single kubernetes node.
When a node is added to the cluster, a new pod will be started automatically.
Same, when a node is removed, the pod will not be rescheduled on another node.

###### Resource Usage Monitoring (deprecated)
Heapster enables Container Cluster Monitoring and Performance Analysis.
It's providing a monitoring platform for Kubernetes.
Heapster exports clusters metricks via REST endpoints.
You can use different backends with Heapster.

###### Autoscaling
Kubernetes has the possibility to automatically scale pods based on metrics.
Kubernetes can automatically scale a Deployment, Replication Controller or replicaSet.
In Kubernetes 1.3 scalling based on CPU usage is possile out of the box.
Autoscaling will use heapster, the monitoring tool, to gather its metrics and make scaling decisions.

```bash
kubectl create -f k8s/autoscaling/hpa-example.yml 
kubectl get hpa
kubectl run -i --tty load-generator --image=busybox /bin/sh
while true; do wget -q -0- <http>; done
kubectl get hpa
kubectl get pod

```
Example of Autoscaling

###### Affinity, anti-affinity
Affinity, anti-affinity feature allows to do more complex scheduling then the nodeSelector and also works on pods.
The rules are not hard requirements, bu rather a prefferred rule.
For example, a rule that makes sure 2 different pods will never be on the same node.

###### Custom Resource Definitions
Lets extend the Kubernetes API.
Resources are the endpoints in the Kubernetes API that store collections of API Objects.
For example, there is the built-in Deployment resource, that can be used to deploy applications.
In the yaml files you describe the object, using the Deployment resource type.
You crete the object on the cluster by using kubectl.
Operators, exmplained in the next lecture, use these CRDs to extend the Kubernets API, with their own functionality.

###### Operators
Is a method of packaging, deploying and managing a Kubernetes application.
An operator contains a lot of the management logic that you as an administrator or user might want, rather than having to implement it yourself.

###### Namespaces
Namespaces allow to create virtual clusters within the same physical cluser.
Namespaces logically separates you cluster.
The standard namespace is called default and that's where all resources are launched in by default.
There is also namespace for kubernetes specific resources, called kube-system.
We can also limit objects (secrets, services, configmaps).
```bash 
kubectl get namespaces
```
List namespaces

```bash 
kubectl create -f k8s/resourcequotas/resourcequota.yml
kubectl create -f k8s/resourcequotas/helloworld-no-quotas.yml
kubectl get deploy --namespace=myspace
kubectl describe rs/<deploy> --namespace=myspace
kubectl create -f k8s/resourcequotas/helloworld-with-quotas.yml
```
Example of Namespaces

###### Networking
Container to container communication  within a pod through localhost and the port number.
Pod to Service communication using nodePort using DNS.
External to service communication using loadbalancer, ingress, nodePort.
Pod to pod communication.
Kubernetes assumes that pods should be able to communicate to other pods, regardless of which node thay are running. 
Every pod has its own IP address. Pods on different nodes need to be able to communicate to each other using those IP addresses.
Not every cloud providers has VPC. Thre are alternatives:
- Container Network Interface (CNI).
Software that provides libraries/ plugins for network interface within containers eg. Calico, Weave.
- An Overlay Network eg. Flannel

###### Node Maintenance
It's the Node Controller that is responsible for managing the Node objects.
It assigns IP sapce to the node when a new node is launched.
Its keeps the node list  up to date with the avaialable machines.
The node controller is also monitoring the health of the node.
If a node is unhealthy it get deleted
Pods running on the unhealthy node will theh get rescheduled.
Node Maintenance
When addid a new node, the kubelet will attempt to register itself.
This is called self-registration and is the default behavior.
It allows to easily add more nodes to the cluster without making API changes.


###### Components
- Envoy: Sidecar proxy per microservice that handles inbound/outbound traffic within each Pod
- Gateway: Inbound gateways (Ingress) and Output gateway (Engress)
- Mixer: Policy/precondition check and telemetry
- Pilot: Converts high level routing rules that control traffic behaviour into Envoy-specific configurations, propagates them to the sidecar at runtime
- Citadel: Certificate Authority for service-to-service auth and encryption



###### Helm
Helm is a single binary that manages deploying Charts to Kubernetes. A chart is a packaged unit of kubernetes software(collection of files that describe a set of Kubernetes reources).
A single chart can deploy an app, a piece of software, or database.
Chart uses tempaltes, that are typically developed by a package maintainer.
Then will generate yaml files that  Kubernetes understands.
Helm allows to do upgrades and rollbacks
```bash
helm init 
helm install
helm search 
helm list
helm upgrade
helm rollback
```

```bash
helm create mychart
```
will create Chart.yaml, values.yaml, templates/

###### Serverless
Public Cloud providers often provide Serverless capabilities in which you can  deploy functions, rather than instances or containers.
- Azure functions
- AWS Labmda
- Google Cloud Functions
With these products, you don't need to manager the underlaying infrastacture.
The functions are also not "always-on" unlike containers and instances, which can greatly reduce the cost of serverless if the function doesn't need to be executed a lot.
Serverless in public cloud can reduce the complexity, operational coasts, and engineering time to get code running.
- OpenFaas
- Kubeless
- Fission
- OpenWhisk
You can install and use any of the projects to let developers functions on Kubernetes.

###### Kubeless
Kubeless is a Kubernetes native framerowk, it laverages the Kubernetes resources to provide auto-scaling, API routing, monitoring etc.
It uses Custom Resource Definitions to be able to create functions.
Once you deployed your function, you'll need to determine how it'll be triggered.
```bash
wget https://github.com/kubeless/kubeless/releases/download/v1.0.4/kubeless_linux-amd64.zip
unzip kubeless_linux-amd64.zip
sudo mv bundles/kubeless_linux-amd64/kubeless /usr/local/bin
rm -r bundles/
kubectl create ns kubeless
kubectl create -f https://github.com/kubeless/kubeless/releases/download/v1.0.4/kubeless-v1.0.4.yaml
kubeless function deploy hello --runtime python2.7 \
                              --from-file k8s/kubeless/python-example/example.py \
                              --handler test.hello
kubeless function deploy myfunction --runtime nodejs6 \
                              --dependencies node-example/package.json \
                              --handler test.myfunction \
                              --from-file k8s/kubeless/node-example/example.js

kubeless function ls
kubeless function call myfunction --data 'This is some data'
kubeless logs <pod_name>
```

```bash
kubectl create -f nginx-ingress-controller-with-elb.yml
kubeless trigger http create myfunction --function-name myfunction --hostname myfunction.kubernetes.newtech.academy
kubectl get svc -n ingress-nginx -o wide
kubectl get ingresss
```


###### Istio
```bash
wget https://github.com/istio/istio/releases/download/1.2.4/istio-1.2.4-linux.tar.gz
tar xzf istio-1.2.4-linux.tar.gz
kubectl apply -f istio-1.2.4/install/kubernetes/hem/istio/templates/crds.yaml
kubectl get crds
kubectl apply -f istio-1.2.4/install/kubernetes/istio-demo.yaml
kubectl get pod -n isto-system
kubectl apply -f <(istioctl kube-inject -f k8s/istio/helloworld.yaml)
kubectl get pods
kubectl apply -f k8s/istio/helloworld-gw.yaml 
```
Install Istio 1.2.4



```bash
curl -L https://istio.io/downloadIstio | sh -
cd istio-ABC
for i in install/kubernetes/helm/istio-init/files/crd*yaml; do kubectl apply -f $i; done
kubectl apply -f install/kubernetes/istio-demo.yaml
kubectl get namespaces
kubectl get pods -n istio-system
kubectl get svc -n istio-system
# istio-ingressgateway has already been assigned with externall IP address
```
Install Istio 1.5.2

The goals of Istio are:
- Retries
- Canary Deployments
- Security by default: no changes needed for application code and infrastucture.
- Defence in depth: integrate with existing security systems to provide multiple layers of defence.
- Zero-trust network: build security solutions on untrusted networks.

Istio provides two types of authentication:
- Transport authentication (service to service authentication) using mutial TLS.
- Origin authentication (end-end authentication) using JSON Web Token.

###### Istio Glossary
- Gateway: configures a load balancer for HTTP/TCP traffic, enables ingess traffic into the service mesh (just like Ingress)
- Virual Service: defines the rules that control how requests for a service are routed within the service mesh
- Destination rule: configures the set of policies to be aplied to a request after VirtualService routing has occurred
- Service Version aka Subset allows to select a subset of pods based on labels
- Service Entry enables requests to service outside of the service mesh

![Istio Glossary](/k8s/imgs/istioGlossary.png)


```bash
istioctl verify-install
kubectl  cluster-info
``` 
Verify Istio installation

###### Demo #1
```bash
kubectl create namespace istio-demo
kubectl label namespace istio-demo istio-injection=enabled
# label the namespace so that the istio sidecar is working propery
kubectl apply -f k8s/istio/nginx/nginx-app.yaml -n istio-demo
kubectl get pods -n istio-demo
kubectl  -n istio-demo describe pod
# Nginx and sidecar container (istio-proxy) deployed
kubectl apply -f k8s/istio/nginx/nginx-app-istio-gateway.yaml -n istio-demo
# Deploy gateway for service; allow access nginx if request contains header Host: nginx-app.demo
kubectl apply -f k8s/istio/nginx/nginx-app-istio-virtual-service.yaml -n istio-demo

curl -H "Host: nginx-app.demo" <EXTERNAL-IP>
```
###### Demo #2 Controlling Ingress Traffic
```bash
kubectl label namespace istio-demo istio-injection=enabled
kubectl get svc istio-ingressgateway -n istio-system
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo $INGRESS_HOST
cd k8s/istio/hello
kubectl apply -f hello-istio.yaml -n istio-demo
kubectl apply -f hello-istio-gateway.yaml -n istio-demo
kubectl apply -f hello-istio-virtual-service.yaml -n istio-demo
kubectl get all
watch -n 1 -d http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```
###### Demo #3 Path Based Routing

```bash
cd k8s/istio/hello
kubectl apply -f hello-istio-destination.yaml
# apply the version subsets as destinations

kubectl apply -f hello-istio-uri-match.yaml
# apply path based routing

http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
http get $INGRESS_HOST/api/v1/hello Host:hello-istio.cloud
http get $INGRESS_HOST/api/v2/hello Host:hello-istio.cloud
```
###### Demo #4 Header Based Routing

```bash
cd k8s/istio/hello
kubectl apply -f kubernetes/hello-istio-user-agent.yaml
http get $INGRESS_HOST/api/hello User-Agent:Chrome Host:hello-istio.cloud
# Reach v2

http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
# Reach v1

kubectl apply -f kubernetes/hello-istio-user-cookie.yaml
http get $INGRESS_HOST/api/hello Cookie:user=packtpub Host:hello-istio.cloud
# Reach v2

http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
# Reach v1
```

###### Demo #5 Weight Based Routing

```bash
cd k8s/istio/hello
kubectl apply -f hello-istio-destination.yaml
# apply the version subsets as destinations

kubectl apply -f hello-istio-50-50.yaml
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```

###### Demo #6 Perform canary release deployment

```bash
cd k8s/istio/hello
kubectl apply -f hello-istio-100-0.yaml
kubectl apply -f hello-istio-75-25.yaml
kubectl apply -f hello-istio-50-50.yaml
kubectl apply -f hello-istio-25-75.yaml
kubectl apply -f hello-istio-0-100.yaml
```

###### Demo #7 Controlling Egress Traffic

Istio has two Egress modes:
1) ALLOW_ANY - default mode
2) REGISTRY_ONLY - explicit trafiic controll

```bash
# get the current egress mode
kubectl get configmap istio -n istio-system -o yaml | grep -o "mode: ALLOW_ANY"
export SOURCE_POD=$(kubectl get pod -l app=hello-istio-console -o jsonpath={.items..metadata.name})
kubectl exec -it $SOURCE_POD -c console /bin/sh
wget -S -q https://www.google.com // this should work
# disable ALLOW_ANY egress mode
kubectl get configmap istio -n istio-system -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | kubectl replace -n istio-system -f -
# Right now pod is unable to connect to outside wold eg. google.com
kubectl exec -it $SOURCE_POD -c console /bin/sh
wget -S -q https://www.google.com // this should't work
kubectl apply -f kubernetes/hello-istio-egress.yaml
kubectl exec -it $SOURCE_POD -c console /bin/sh
wget -S -q https://www.google.com // this should work
```


Service Resilience
Circuit Breaker

###### Demo #8 Setting Request Timeouts
```bash
cd timeout
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-virtual-service.yaml
kubectl apply -f hello-istio-destination.yaml

kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud sleep==3
```

Next, edit the virtual service definitions for `hello-istio` and the `hello-message` service to configure the timeouts.

```yaml
    # configure a 2s timeout
    timeout: 2s
```

Issue the following commands to apply and see the timeouts in action.
```bash
kubectl apply -f hello-istio-virtual-service.yaml
kubectl apply -f hello-message-virtual-service.yaml
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud sleep==3
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud sleep==1
```
###### Demo #9 Connection Pools
```bash
cd connectionpools
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-virtual-service.yaml
kubectl apply -f hello-istio-destination.yaml
kubectl apply -f hello-message-virtual-service.yaml
kubectl apply -f hello-message-destination.yaml


kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```
Next, edit the destination rule definitions for hello-istio and hello-message to configure the connection pools and bulk heads.
```yaml
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100             # maximum number of TCP conns
        connectTimeout: 5s              # TCP connection timeout
        tcpKeepalive:                   # keep alive settings
          time: 3600s
          interval: 60s
      http:
        maxRequestsPerConnection: 25    # max request per keep-alive
        http2MaxRequests: 5             # max number of HTTP2 conns
        http1MaxPendingRequests: 5      # max number of pending reqs
        maxRetries: 3                   # max number of retries
        idleTimeout: 60s                # idle timeout for connection
```
Now apply the modified destination rule definitions for the two services.

```bash
kubectl apply -f hello-istio-connection-pool.yaml
kubectl apply -f hello-message-connection-pool.yaml
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```
###### Demo #10 Retries
```bash
cd retries
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-virtual-service.yaml
kubectl apply -f hello-istio-destination.yaml
kubectl apply -f hello-message-virtual-service.yaml
kubectl apply -f hello-message-destination.yaml

kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```

Next, edit the virtual service definitions for `hello-istio` and  `hello-message` to configure the retry behaviour.

```yaml
    retries:
      attempts: 3
      perTryTimeout: 500ms
      retryOn: gateway-error,connect-failure,refused-stream
```

Now apply the modified destination rule definitions for the two services.

```bash
kubectl apply -f hello-istio-retries.yaml
kubectl apply -f hello-message-retries.yaml
```

###### Demo #11 Injecting HTTP Delay Fault

```bash
cd injectingHTTPDelayFault
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-v1.yaml
kubectl apply -f hello-istio-destination.yaml
kubectl apply -f hello-message-virtual-service.yaml
kubectl apply -f hello-message-destination.yaml
```

First, make sure everything is running correctly without delays.
```bash
kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```

Next, configure a HTTP delay fault for any traffic to the hello-message v1 virtual service.

```yaml
- fault:
    delay:
      percentage:
        value: 100.0
      fixedDelay: 5s
  # optionally add header match here
  route:
  - destination:
      host: hello-message
      subset: v1
```

Apply the modified virtual service and check that the HTTP delay is configured correctly.

```bash
kubectl apply -f hello-message-v1-delay.yaml

# you should see an error message after 3s delay -> timeout working
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```


###### Demo #12 Injecting HTTP Abort Fault
```bash
cd injectingHTTPAbortFault
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-v1.yaml
kubectl apply -f hello-istio-destination.yaml
kubectl apply -f hello-message-virtual-service.yaml
kubectl apply -f hello-message-destination.yaml
```

First, make sure everything is running correctly without any faults.
```bash
kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```

Next, configure a HTTP abort fault for any traffic to the hello-message v1 virtual service.

```yaml
- fault:
    abort:
      httpStatus: 500
      percentage:
        value: 100.0
  # optionally add header match here
  route:
  - destination:
      host: hello-message
      subset: v1
```

Apply the modified virtual service and check that the HTTP abort fault is configured correctly.

```bash
kubectl apply -f hello-message-v1-abort.yaml

# you should see an error message about the abort filter
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
```

###### Demo 13 Envoy Filters
```bash
cd envoyFilters
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-v1.yaml
kubectl apply -f hello-istio-destination.yaml

kubectl apply -f hello-message-virtual-service.yaml
kubectl apply -f hello-message-destination.yaml
```

First, make sure everything is running correctly.

```
kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

# check the container logs of the hello-message v1 pod
kubectl logs hello-message-v1-6dcc4fff9-hnbxs -c hello-message
```

Apply the prepared envoy filter manifest and check that everything is working as expected. It does take a while for the sidecar to pick up the new filter configuration.

```yaml
apiVersion: networking.istio.io/v1alpha3
kind: EnvoyFilter
metadata:
  name: hello-message-lua
spec:
  workloadSelector:
    labels:
      app: hello-message
  configPatches:
    # adds the lua filter to the listener/http connection manager
    # see https://istio.io/docs/reference/config/networking/envoy-filter/
    # see https://www.envoyproxy.io/docs/envoy/latest/configuration/http/http_filters/lua_filter#
  - applyTo: HTTP_FILTER
    match:
      context: SIDECAR_INBOUND
      listener:
        filterChain:
          filter:
            name: "envoy.http_connection_manager"
            subFilter:
              name: "envoy.router"
    patch:
      operation: INSERT_BEFORE
      value:
       name: envoy.lua
       config:
         inlineCode: |
           function envoy_on_request(request_handle)
             -- send back static response and do not continue
             request_handle:respond({[":status"] = "200"}, "Envoy Filtered Message")
           end

           function envoy_on_response(request_handle)
             -- add response specific logic here
           end
```

```
kubectl apply -f kubernetes/hello-message-v1-filter.yaml
watch -n 1 -d http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
kubectl logs hello-message-v1-6dcc4fff9-hnbxs -c hello-message
```

###### Demo 14 Traffic Mirroring
```bash
cd trafficMirroring
kubectl apply -f hello-istio.yaml
kubectl apply -f hello-istio-gateway.yaml
kubectl apply -f hello-istio-v1.yaml
kubectl apply -f hello-istio-destination.yaml

kubectl apply -f hello-message-virtual-service.yaml
kubectl apply -f hello-message-destination.yaml
```
First, make sure everything is running correctly.
```bash
kubectl get all
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

# check the container logs of the hello-message pod
kubectl logs hello-message-v2-6dcc4fff9-hnbxs -c hello-message
```

Next, configure traffic mirroring for the hello-message v2 virtual service.

```yaml
  mirror:
    host: hello-message
    subset: v2
  mirror_percent: 100
```

Apply the changes to the virtual service, invoke the service and finally check the container logs.

```bash
kubectl apply -f hello-message-v2-mirroring.yaml

http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

kubectl logs hello-message-v2-6dcc4fff9-hnbxs -c hello-message
```

###### Demo 15 Mutual TLS between services

Make sure you have installed the `demo` profile for Istio.

```bash
cd mtls
istioctl manifest apply --set profile=demo

kubectl get svc istio-ingressgateway -n istio-system
```

We will be using two different namespaces to demonstrate the security mTLS features.

```bash
kubectl create namespace hello-istio
kubectl label namespace hello-istio istio-injection=enabled

# The default namespace will not have the sidecar injection
kubectl label namespace default istio-injection=disabled
```

Next, we will apply the demo set of microservices. Per default, the mTLS between services is in PERMISSIVE mode, meaning that encrypted and unencrypted traffic is allowed in the service mesh.

```bash
kubectl apply -f demo/

kubectl apply -f hello-istio-secure.yaml
kubectl apply -f hello-istio-insecure.yaml

kubectl get all -n hello-istio
kubectl get all -n default
```

First, we check that we can call a service from the secure namespace and console pod:
```bash
kubectl exec -n hello-istio -c console -it hello-istio-secure-..... /bin/sh

wget hello-istio:8080/api/hello -S -O - | more
wget hello-istio.hello-istio.svc.cluster.local:8080/api/hello -S -O - | more
```

Next, we check that we can also call the service from an insecure namespace and console pod:
```bash
kubectl exec -it hello-istio-insecure-6969cf44bf-..... /bin/sh

wget hello-istio.hello-istio.svc.cluster.local:8080/api/hello -S -O - | more
```

###### Demo 15 Authorization on Ingress Gateway

```bash
cd authorizationOnIngressGateway
kubectl create namespace hello-istio
kubectl label namespace hello-istio istio-injection=enabled

kubectl apply -f demo/
kubectl get all -n hello-istio

kubectl get svc istio-ingressgateway -n istio-system
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

First, make sure that you can call the services without any applied `AuthorizationPolicy` and
that you have configured to the gateway to forward source IP addresses.

```bash
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

kubectl patch svc istio-ingressgateway -n istio-system -p '{"spec":{"externalTrafficPolicy":"Local"}}'
```

Next, find out your client IP address and issue the following commands to `DENY` or `ALLOW` any traffic entering the ingress gateway from your IP.

```bash
curl -s 'https://api.ipify.org?format=json'
export CLIENT_IP=$(curl -s 'https://api.ipify.org?format=text')

sed "s/<<CLIENT_IP>>/$CLIENT_IP/" hello-istio-gateway-policy-deny.yaml | kubectl apply -f -
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

sed "s/<<CLIENT_IP>>/$CLIENT_IP/" hello-istio-gateway-policy-allow.yaml | kubectl apply -f -
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

# do some cleanup
kubectl delete AuthorizationPolicy -n istio-system --all
```

###### Demo 16 Authorization for HTTP Traffic
```bash
cd authorizationForHTTPTraffic
kubectl create namespace hello-istio
kubectl label namespace hello-istio istio-injection=enabled

kubectl apply -f demo/

kubectl apply -f hello-istio-secure.yaml
kubectl apply -f hello-istio-insecure.yaml

kubectl get svc istio-ingressgateway -n istio-system
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

Any HTTP traffic inside the service mesh can be enabled to disabled using an `AuthorizationPolicy`.

```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: hello-message-http-policy
  namespace: hello-istio
spec:
  selector:
    matchLabels:
      app: hello-message
  # toggle between DENY and ALLOW
  action: DENY
  rules:
  - to:
      - operation:
          methods: ["GET"]
    from:
      - source:
          namespaces:
            - "hello-istio"
```

Apply the above policy and check that the HTTP communication within the `hello-istio` namespace is not possible anymore but it is possible from outside the namespace.

```bash
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud
kubectl apply -f hello-message-http-policy.yaml
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

kubectl exec -it hello-istio-insecure-6969cf44bf-.... /bin/sh
wget hello-message.hello-istio.svc.cluster.local:8080/api/message/hello -S -O - | more

# do some cleanup
kubectl delete AuthorizationPolicy -n hello-istio --all
```

###### Demo 17 Authorization with JWT
```bash
cd authorizationWithJWT
kubectl create namespace hello-istio
kubectl label namespace hello-istio istio-injection=enabled

kubectl apply -f demo/

kubectl apply -f hello-istio-secure.yaml
kubectl apply -f hello-istio-insecure.yaml

kubectl get svc istio-ingressgateway -n istio-system
export INGRESS_HOST=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

Optionally, if you want to experiment with your own JWT and JWKS then follow these instructions to create your own credentials.

```bash
cd data/

# download the JWT generator from the Istio repository
$ wget https://raw.githubusercontent.com/istio/istio/release-1.5/security/tools/jwt/samples/gen-jwt.py

# (optionally) create your own key
openssl genrsa -out key.pem 2048

# generate a new JWKS and JWT data set
pip3 install jwcrypto
python3 gen-jwt.py key.pem --iss packtpub --sub demo --aud students --jwks=./jwks.json --expire=3153600000 --claims=publisher:packtpub > packtpub.jwt
```

Here, we will apply the JWT authentication and authorization policies to the Istio service mesh. Without the policies, everything should work as expected. Once we have applied the policies, all requests without a JWT are denied with a `403 Forbidden`. Only requests with the correct JWT bearer token are accepted.

```bash
# every call works
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

# all requests without JWT are denied with 403
kubectl apply -f hello-istio-jwt-authz.yaml
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud

# prepare the JWT Bearer token
export BEARER_TOKEN="Bearer $(cat data/packtpub.jwt)"
echo $BEARER_TOKEN

# a request with correct JWT bearer token is accepted
http get $INGRESS_HOST/api/hello Host:hello-istio.cloud Authorization:$BEARER_TOKEN
```

###### Kustomize
Kustomize lets you customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.
In order to use Kustomize we have to use '-k' flag.
```bash
cd k8s/kustomize/base
kubectl apply -k .
```

```bash
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-7cf5c7ddb5-f84zf   1/1     Running   0          10s

NAME                 TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.43.X.Y     <none>        443/TCP        123m
service/nginx        NodePort    10.43.X.Y   <none>        80:30523/TCP   10s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           10s
```

```bash
cd ../overlay/staging
kubectl apply -k .
```

```bash
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-7cf5c7ddb5-f84zf          1/1     Running   0          5m1s
pod/staging-nginx-f4b4bbfc8-khwfm   1/1     Running   0          6s

NAME                    TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.43.X.Y     <none>        443/TCP        128m
service/nginx           NodePort    10.43.X.Y   <none>        80:30523/TCP   5m1s
service/staging-nginx   NodePort    10.43.X.Y   <none>        80:31543/TCP   6s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx           1/1     1            1           5m1s
deployment.apps/staging-nginx   1/1     1            1           6s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-7cf5c7ddb5          1         1         1       5m1s
replicaset.apps/staging-nginx-f4b4bbfc8   1         1         1       6s
```
```bash
cd ../production
kubectl apply -k .
```

```bash
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-7cf5c7ddb5-f84zf          1/1     Running   0          6m34s
pod/staging-nginx-f4b4bbfc8-khwfm   1/1     Running   0          99s
pod/prod-nginx-59b47887cb-xm6d8     1/1     Running   0          24s
pod/prod-nginx-59b47887cb-nx7pq     1/1     Running   0          24s
pod/prod-nginx-59b47887cb-552jt     1/1     Running   0          24s

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.43.X.Y      <none>        443/TCP        129m
service/nginx           NodePort    10.43.X.Y    <none>        80:30523/TCP   6m34s
service/staging-nginx   NodePort    10.43.X.Y    <none>        80:31543/TCP   99s
service/prod-nginx      NodePort    10.43.X.Y   <none>        80:30339/TCP   25s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx           1/1     1            1           6m34s
deployment.apps/staging-nginx   1/1     1            1           99s
deployment.apps/prod-nginx      3/3     3            3           25s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-7cf5c7ddb5          1         1         1       6m34s
replicaset.apps/staging-nginx-f4b4bbfc8   1         1         1       99s
replicaset.apps/prod-nginx-59b47887cb     3         3         3       25s
```

```bash
cd ../ingress/single
kubectl apply -k .
```

```bash
NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-7cf5c7ddb5-f84zf          1/1     Running   0          58m
pod/staging-nginx-f4b4bbfc8-khwfm   1/1     Running   0          53m
pod/prod-nginx-59b47887cb-xm6d8     1/1     Running   0          52m
pod/prod-nginx-59b47887cb-nx7pq     1/1     Running   0          52m
pod/prod-nginx-59b47887cb-552jt     1/1     Running   0          52m
pod/dev-nginx-6cc6b9f788-sq9z4      1/1     Running   0          5s

NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes      ClusterIP   10.43.X.Y       <none>        443/TCP        3h1m
service/nginx           NodePort    10.43.X.Y     <none>        80:30523/TCP   58m
service/staging-nginx   NodePort    10.43.X.Y     <none>        80:31543/TCP   53m
service/prod-nginx      NodePort    10.43.X.Y    <none>        80:30339/TCP   52m
service/dev-nginx       ClusterIP   10.43.X.Y   <none>        80/TCP         5s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx           1/1     1            1           58m
deployment.apps/staging-nginx   1/1     1            1           53m
deployment.apps/prod-nginx      3/3     3            3           52m
deployment.apps/dev-nginx       1/1     1            1           5s

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-7cf5c7ddb5          1         1         1       58m
replicaset.apps/staging-nginx-f4b4bbfc8   1         1         1       53m
replicaset.apps/prod-nginx-59b47887cb     3         3         3       52m
replicaset.apps/dev-nginx-6cc6b9f788      1         1         1       5s
```


###### Kops (Kubernetes Operations)
Tool used to spin up a highly available production cluster.
Used to setup Kubernetes on AWS.
Works only on Mac / Linux.