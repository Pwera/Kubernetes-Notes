# Kubernetes
<i>Kubernetes</i> is and open source orchestration system for containers. It lets you schedule containers on a cluster of machines. Kubernetes can run multiple contianers on one machine. Kubernetes will manage the state of these containers. Can start the contaienr on specific nodes. Will restart container when it gets killed. Can move containers from one node to another node. Containers inside same pod can communicate with localhost.

<i>Kubelets</i> are responsible to launch the pods. It's going to connect to the master node to get this information.

<i>Kube-proxy</i> is going to feed its information about what pods are on nodes to iptables. Iptables is the firewall in Linux and it can also route traffic.
So whenever a new pod is launched the kube-proxy is going to change the Iptables rules to make sure that the pod is routable within the cluster.

###### Minikube

<details>
<summary>Click to expand Minikube!</summary>
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

</details>

###### Pod
<details>
<summary>Click to expand Pod!</summary>
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
</details>

###### Service

<details>
<summary>Click to expand Service!</summary>
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

</details>

###### Scaling pods

<details>
<summary>Click to expand Scaling pods!</summary>
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

</details>

###### Deployments

<details>
<summary>Click to expand Deployments!</summary>

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

Deploymnet field
observedGeneration:
Shows how often the deployment has been updated. This information can be used to understand the rollout and rollback situation of the deployment.

</details>

###### Services

<details>
<summary>Click to expand Services!</summary>

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
</details>

###### Labels

<details>
<summary>Click to expand Labels!</summary>

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
</details>

###### Health checks

<details>
<summary>Click to expand Health checks!</summary>
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
</details>

###### Readiness Probe.

<details>
<summary>Click to expand Readiness Probe!</summary>
Besides <i>livenessProbes</i> you can also use <i>readinessProbe</i> on a container within a Pod.
<i>LivenessProbes</i> indicate wheter container is running.
If the check fail, the container will be restarted.
<i>ReadinessProbe</i> indicateswhether the container is ready to serve requests.
If the check fails the container will not be restarted, but the Pod's IP address will be removed ffom the service, so it'll not serve any requesty anymore.

Using a defined header to a particular port and path, the container is not considered healthy until the web server returns a code 200-399. Any other code indicates failure, and the probe will try again.

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
</details>

###### Pod state
<details>
<summary>Click to expand Pod state!</summary>
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
</details>

######  Secrets
<details>
<summary>Click to expand Secrets!</summary>
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

In order to encrypt secrets, you must create an EncryptionConfiguration object with a key and proper identity. Then, the kube-apiserver needs the --encryption-provider-config flag set to a previously configured provider, such as aescbc or ksm. Once this is enabled, you need to recreate every secret, as they are encrypted upon write. Multiple keys are possible. Each key for a provider is tried during decryption. The first key of the first provider is used for encryption. To rotate keys, first create a new key, restart (all) kube-apiserver processes, then recreate every secret.



There is no limit to the number of Secrets used, but there is a 1MB limit to their size. Each secret occupies memory, along with other API objects, so very large numbers of secrets could deplete memory on a host.

They are stored in the tmpfs storage on the host node, and are only sent to the host running Pod. All volumes requested by a Pod must be mounted before the containers within the Pod are started. So, a secret must exist prior to being requested.
</table>
</details>

######  Service Discovery using DNS.
<details>
<summary>Click to expand Service Discovery using DNS!</summary>
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

</details>

######  ConfigMap
<details>
<summary>Click to expand ConfigMap!</summary>
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
</details>

######  Ingress
<details>
<summary>Click to expand Ingress!</summary>
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
###### External DNS
On public cloud providers, you can use the ingress controller to reduce the cost of LoadBalancer.
You can use one LB that capures all the external traffic and send it to the ingress controller.
The ingress controller can be configured to route different traffic to your apps based on http rules(host and prefixes).
This only works for https- based applications.
</details>


###### Egress Gateway
<details>
<summary>Click to expand Egress Gateway!</summary>
Every traffic that needs to exit the service mesh needs to go via Egress Gateway.
</details>


###### Volumes
<details>
<summary>Click to expand Volumes!</summary>
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
<td>NFS example:
<td>

``` bash
sudo apt-get update && sudo apt-get install -y nfs-kernel-server

sudo mkdir /opt/sfw

sudo chmod 1777 /opt/sfw/

sudo bash -c "echo software > /opt/sfw/hello.txt"

sudo bash -c "echo '/opt/sfw/ *(rw,sync,no_root_squash,subtree_check)' >> /etc/exports"

sudo exportfs -ra

echo
echo "Should be ready. Test here and second node"
echo

sudo showmount -e localhost
```
Commands on second node:
```
sudo apt-get -y install nfs-common nfs-kernel-server
showmount -e master
sudo mount master:/opt/sfw /mnt
#unless you edit/etc/fstabthis is not a persistent mount.
```
<tr>
<td> PV Manifest:
<td>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pvvol-1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /opt/sfw
    server: master
    readOnly: false
```
</table>


A particular access mode is part of a Pod request. As a request, the user may be granted more, but not less access, though a direct match is attempted first. The cluster groups volumes with the same mode together, then sorts volumes by size, from smallest to largest. The claim is checked against each in that access mode group, until a volume of sufficient size matches. The three access modes are RWO (ReadWriteOnce), which allows read-write by a single node, ROX (ReadOnlyMany), which allows read-only by multiple nodes, and RWX (ReadWriteMany), which allows read-write by many nodes.


Other Volume Types
 GCEpersistentDisk, awsElasticBlockStore, nfs, azureDisk, azureFile, csi, downwardAPI, fc (fibre channel), flocker, gitRepo, local, projected, portworxVolume, quobyte, scaleIO, secret, storageos, vsphereVolume, persistentVolumeClaim, etc.​

 NFS (Network File System) and iSCSI (Internet Small Computer System Interface) are straightforward choices for multiple readers scenarios.


The StorageClass API resource allows an administrator to define a persistent volume provisioner of a certain type, passing storage-specific parameters.


###### Volumes AutoProvisioning
The Kubernetes plugins have the capability to provision storage.
This is done using the StorageClass object.
</details>

###### Pod Presets
<details>
<summary>Click to expand Pod Presets!</summary>
Can inject information into pods at runtime.
Pod Presets are used to inject Kubernetes resources like Secrets, ConfigMaps, Volumes and environmental varibles.
When injecting environment variables and volumemounts, the pod presets will apply the changes to all containers within the pod.

<table>
<td>PodPreset<br>definition
<td>

```yaml
apiVersion: settings.k8s.io/v1alpha1
kind: PodPreset
metadata:
  name: allow-database
spec:
  selector:
    matchLabels:
      role: frontend
  env:
    - name: DB_PORT
      value: "6379"
  volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
    - name: cache-volume
      emptyDir: {}
```
<tr>
<td>Attaching<br>PodPreset<br>to Pod
<td>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: website
  labels:
    app: website
    role: frontend
spec:
  containers:
    - name: website
      image: nginx
      ports:
        - containerPort: 80
```

<tr>
<td>Apply<br>Objects
<td>

```bash
kubectl create -f k8s/pod-presets/pod-presets.yml
kubectl get podpresets
kubectl create -f deployments.yml
```
</table>
</details>

###### StatefulSets
<details>
<summary>Click to expand StatefulSets!</summary>
It's introduced to be able to run stateful applications.
A StatefulSet will allow stateful app to use DNS to find other peers.
```bash
kubectl create -f k8s/statefulset/cassandra.yaml
kubectl get pod
kubectl get pv
watch kubectl get pod
```
</details>


###### DeamonSets
<details>
<summary>Click to expand DeamonSets!</summary>
Deamon Sets ensure that every single node in the Kubernetes cluster runs that same pod resource.
This is useful if you want to ensure that a certain pod is running on every single kubernetes node.
When a node is added to the cluster, a new pod will be started automatically.
Same, when a node is removed, the pod will not be rescheduled on another node.
</details>

###### Resource Usage Monitoring (deprecated)
<details>
<summary>Click to expand Resource Usage Monitoring!</summary>
Heapster enables Container Cluster Monitoring and Performance Analysis.
It's providing a monitoring platform for Kubernetes.
Heapster exports clusters metricks via REST endpoints.
You can use different backends with Heapster.
</details>

###### Autoscaling
<details>
<summary>Click to expand Autoscaling!</summary>
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
</details>

###### Affinity, anti-affinity
<details>
<summary>Click to expand Affinity, anti-affinity!</summary>
Affinity, anti-affinity feature allows to do more complex scheduling then the nodeSelector and also works on pods.
The rules are not hard requirements, bu rather a prefferred rule.
For example, a rule that makes sure 2 different pods will never be on the same node.
</details>

###### Custom Resource Definitions
<details>
<summary>Click to expand Custom Resource Definitions!</summary>
Lets extend the Kubernetes API.
Resources are the endpoints in the Kubernetes API that store collections of API Objects.
For example, there is the built-in Deployment resource, that can be used to deploy applications.
In the yaml files you describe the object, using the Deployment resource type.
You crete the object on the cluster by using kubectl.
Operators, exmplained in the next lecture, use these CRDs to extend the Kubernets API, with their own functionality.
</details>

###### Operators
<details>
<summary>Click to expand Operators!</summary>
Is a method of packaging, deploying and managing a Kubernetes application.
An operator contains a lot of the management logic that you as an administrator or user might want, rather than having to implement it yourself.
</details>

###### Namespaces
<details>
<summary>Click to expand Namespaces!</summary>
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
</details>

###### Networking

<details>
<summary>Click to expand Networking!</summary>
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
</details>




###### Pod Security Policies
<details>
<summary>Click to expand Pod Security Policies!</summary>
A PodSecurityPolicy is a Kubernetes API object. You can create them without any modifications to Kubernetes. However, the policies created are not enforced by default.

Kubernetes, by default, allows anything capable of creating a Pod to run a fairly privileged container that can compromise a system. Pod Security Policies protect clusters from privileged pods by ensuring the requester is authorized to create a pod as configured.

To automate the enforcement of security contexts, you can define PodSecurityPolicies (PSP). A PSP is defined via a standard Kubernetes manifest following the PSP API schema.
These policies are cluster-level rules that govern what a pod can do, what they can access, what user they run as, etc.

For instance, if you do not want any of the containers in your cluster to run as the root user, you can define a PSP to that effect. You can also prevent containers from being privileged or use the host network namespace, or the host PID namespace.

While PSP has been helpful, there are other methods gaining popularity. The Open Policy Agent (OPA), often pronounced as "oh-pa", provides a unified set of tools and policy framework. This allows a single point of configuration for all of your cloud deployments.

OPA can be deployed as an admission controller inside of Kubernetes, which allows OPA to enforce or mutate requests as they are received. Using the OPA Gatekeeper it can be deployed using Custom Resource Definitions.

For Pod Security Policies to be enabled, you need to configure the admission controller of the controller-manager to contain PodSecurityPolicy. These policies make even more sense when coupled with the RBAC configuration in your cluster. This will allow you to finely tune what your users are allowed to run and what capabilities and low level privileges their containers will have.

<table>

<td> PodSecurityPolicy
<td>

``` yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restrictive
spec:
  privileged: false
  hostNetwork: false
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  hostPID: false
  hostIPC: false
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - 'configMap'
  - 'downwardAPI'
  - 'emptyDir'
  - 'persistentVolumeClaim'
  - 'secret'
  - 'projected'
  allowedCapabilities:
  - '*'
```

</table>

<h3>Admission Controller</h3>
Admission controllers intercept requests to the kube-apiserver. The interception occurs before a requested object is persisted but after the request is authenticated and authorized. This enables us to see who or what the requested object came from and validate whether what's being asking for is appropriate. Admission controllers are enabled by adding them to the kube-apiserver flag --enable-admission-plugins. Prior to 1.10, the order of admission controllers mattered when using the, now deprecated, --admission-control flag.

Add the PodSecurityPolicy to the --enabled-admission-plugins flag on the kube-apiserver.

```
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,DefaultTolerationSeconds,MutatingAdmissionWebhook,ValidatingAdmissionWebhook,ResourceQuota,PodSecurityPolicy
```

PodSecurityPolicy has been appended to that list above. Now that Pod Security Policies are enforced and our cluster is absent of policies, new pod creations (including re-creating pods from a scheduling event) will fail.

Create an nginx deployment.

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
```

Check the available pods, replicasets, and deployments in the namespace. Then delete the deployment.

``` yaml
kubectl get po,rs,deploy

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-811c4cff45   1         0         0       9s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-hostnetwork-deployment   0/1     0            0           9s
kubectl delete deploy nginx-deployment
```

This demonstrates that the deployment and replicaset were created but the pod could not be created by the replicaset controller. This is where service accounts come in.

<h4>Service Accounts: Controller Manager<h4>

Pods are rarely created by users. Typically a user creates a Deployment, StatefulSet, Job, or Daemonset. This in turn relies on a controller to create the pod. With this in mind, the kube-controller-manager should be configured to use individual service accounts for each controller it contains.

This can be accomplished by adding the following flag to the command args.

```
--use-service-account-credentials=true
```

This flag is the default for most installers and tooling such as kubeadm.

When the kube-controller-manager starts with this flag, it'll make use of the following service accounts, automatically generated by Kubernetes.

```
kubectl get sa -n kube-system
```

```
attachdetach-controller
certificate-controller
clusterrole-aggregation-controller
cronjob-controller
daemon-set-controller
deployment-controller
disruption-controller
endpoint-controller
job-controller
namespace-controller
node-controller
pv-protection-controller
pvc-protection-controller
replicaset-controller
replication-controller
resourcequota-controller
service-account-controller
service-controller
statefulset-controller
ttl-controller
```

<table>
<td>Start by creating ClusterRoles that allow the use of the restrictive policy and permissive policy.
<td>

``` yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-restrictive
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - restrictive
  verbs:
  - use
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-permissive
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - permissive
  verbs:
  - use
  ```
<tr>
<td>With the ClusterRoles in place, let's start by creating access to use the “default” restrictive policy.

Create a ClusterRoleBinding that grants the psp-restrictive ClusterRole to all controller (system) service accounts.

<td>

``` yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-default
subjects:
- kind: Group
  name: system:serviceaccounts
  namespace: kube-system
roleRef:
  kind: ClusterRole
  name: psp-restrictive
  apiGroup: rbac.authorization.k8s.io
```

<tr>
<td>Create the nginx deployment again.

<td>

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
```

<tr>
<td>
Get the pods, replicasets, and deployments for the namespace.

<td>

```
kubectl get po,rs,deploy

pod/nginx-hostnetwork-deployment-7c74c7d654-tl4v4   1/1     Running   0          3s

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-7c74c7d654   1         1         1       3s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-hostnetwork-deployment   1/1     1            1           3s
```

<tr>
<td>Delete the nginx deployment.
<td>

```
kubectl delete deploy nginx-deployment
```
<tr>
<td>Create the nginx-deployment with 

``` hostNetwork: true ```.

<td>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hostnetwork-deployment
  namespace: default
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
      hostNetwork: true
```
<tr>
<td>Get the pods, replicasets, and deployments for the namespace.
<td>

```
kubectl get po,rs,deploy

NAME                                                            DESIRED   CURRENT   READY   AGE
replicaset.extensions/nginx-hostnetwork-deployment-597c4cff45   1         0         0       9s

NAME                                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/nginx-hostnetwork-deployment   0/1     0            0           9s
```

<tr>
<td>
Here we can see the pod is no longer able to be created by the replicaset.

Describe the replicaset to better understand why it's unable to create the pod.`

<td>


``` yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: psp-permissive
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-permissive
subjects:
- kind: ServiceAccount
  name: daemon-set-controller
  namespace: kube-system
- kind: ServiceAccount
  name: replicaset-controller
  namespace: kube-system
- kind: ServiceAccount
  name: job-controller
  namespace: kube-system
```
<tr>
<td>Now you can create a pod with hostNetwork: true in the kube-system namespace! An example deployment is as follows.
<td>

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hostnetwork-deployment
  namespace: kube-system
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
      hostNetwork: true
```
<tr>
<td>
What if you want to enforce the restrictive policy in a namespace but make an exception for one workload to use the permissive policy? With the current model, we only have cluster-level and namespace-level resolution. To provide workload-level resolution to the permissive policy, we can provide the workload's ServiceAccount the ability to use the psp-permissive ClusterRole.

Create a specialsa ServiceAccount in the default namespace.
<td>

``` yaml
kubectl create serviceaccount specialsa
```

<tr>
<td>Create a RoleBinding in the default namespace binding specialsa to the psp-permissive ClusterRole.
<td>

``` yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: RoleBinding
metadata:
  name: specialsa-psp-permissive
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-permissive
subjects:
- kind: ServiceAccount
  name: specialsa
  namespace: default
```
<tr>
<td>
Create the nginx-deployment in the default namespace with the specialsa service account.
<td>

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-hostnetwork-deployment
  namespace: kube-system
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
      hostNetwork: true
	  serviceAccount: specialsa
```
<tr>
<td>
When the PodSecurityPolicy matches an admission controller to a Pod being requested, it selects that policy and moves on. When debugging, it can be helpful to see which policy was chosen. Kubernetes annotates the pod with the selected PSP so you can see just that.

Search for the psp annotation on the nginx-deployment

<td>

```
kubectl get po $(kubectl get po | egrep -o nginx[A-Za-z0-9-]+) -o yaml | grep -i psp

kubernetes.io/psp: permissive
```

</table>


</details>

###### Node Maintenance
<details>
<summary>Click to expand Node Maintenance!</summary>
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
</details>

###### Container Runtime Interface
<details>
<summary>Click to expand Container Runtime Interface!</summary>
The goal of the Container Runtime Interface (CRI) is to allow easy integration of container runtimes with kubelet. By providing a protobuf method for API, specifications and libraries, new runtimes can easily be integrated without needing deep understanding of kubelet internals.

The project is in early stage, with lots of development in action. Now that Docker-CRI integration is done, new runtimes should be easily added and swapped out. At the moment, CRI-O, rktlet and frakti are listed as work-in-progress.
</details>


###### Container Storage Interface
<details>
<summary>Click to expand Container Storage Interface!</summary>
Adoption of Container Storage Interface (CSI) enables the goal of an industry standard interface for container orchestration to allow access to arbitrary storage systems. Currently, volume plugins are "in-tree", meaning they are compiled and built with the core Kubernetes binaries. This "out-of-tree" object will allow storage vendors to develop a single driver and allow the plugin to be containerized. This will replace the existing Flex plugin which requires elevated access to the host node, a large security concern.
</details>


###### Authentication Authorization Admission Control
<details>
<summary>Click to expand Authentication Authorization Admission Control!</summary>
To perform any action in a Kubernetes cluster, you need to access the API and go through three main steps:

- Authentication (Certificate or Webhook)
- Authorization (RBAC or Webhook)
- Admission Controls.

Admission Control part of the kube-apiserver, which handles and possibly modifies the API requests, asks an outside server, or deny or accept those requests.
Once a request reaches the API server securely, it will first go through any authentication module that has been configured. The request can be rejected if authentication fails or it gets authenticated and passed to the authorization step.

At the authorization step, the request will be checked against existing policies. It will be authorized if the user has the permissions to perform the requested actions. Then, the requests will go through the last step of admission controllers. In general, admission controllers will check the actual content of the objects being created and validate them before admitting the request.

:: Authentication::
The type of authentication used is defined in the kube-apiserver startup options. Below are four examples of a subset of configuration options that would need to be set depending on what choice of authentication mechanism you choose:

```
--basic-auth-file

--oidc-issuer-url

--token-auth-file

--authorization-webhook-config-file

```

One or more Authenticator Modules are used: x509 Client Certs; static token, bearer or bootstrap token; static password file; service account and OpenID connect tokens. Each is tried until successful, and the order is not guaranteed. Anonymous access can also be enabled, otherwise you will get a 401 response. Users are not created by the API, and should be managed by an external system.

::Authorization::
There are three main authorization modes and two global Deny/Allow settings. The three main modes are:

Node
RBAC
Webhook.
They can be configured as kube-apiserver startup options:

```
--authorization-mode=Node,RBAC

--authorization-mode=Webhook
```

The Node authorization is dedicated for kubelet to communicate to the kube-apiserver such that it does not need to be allowed via RBAC. All non-kubelet traffic would then be checked via RBAC.

The authorization modes implement policies to allow requests. Attributes of the requests are checked against the policies (e.g. user, group, namespace, verb).



While RBAC can be complex, the basic flow is to create a certificate for a subject, associate that to a role perhaps in a new namespace, and test. As users and groups are not API objects of Kubernetes, we are requiring outside authentication. After generating the certificate against the cluster certificate authority, we can set that credential for the subject using a context, which is a combination of user name, cluster name, authinfo, and possibly namespace. This information can be seen with the kubectl config get-contexts command.

Roles can then be used to configure an association of apiGroups, resources, and the verbs allowed to them. The user can then be bound to a role limiting what and where they can work in the cluster.

Here is a summary of the RBAC process, typically done by the cluster admin:

Determine or create namespace for the subject
Create certificate credentials for the subject
Set the credentials for the user to the namespace using a context
Create a role for the expected task set
Bind the user to the role
Verify the user has limited access.​



:: Admission Controller ::
The last step in completing an API request is one or more admission controls.

Admission controllers are pieces of software that can access the content of the objects being created by the requests. They can modify the content or validate it, and potentially deny the request.

Admission controllers are needed for certain features to work properly. Controllers have been added as Kubernetes matured. Starting with the 1.13.1 release of the kube-apiserver, the admission controllers are now compiled into the binary, instead of a list passed during execution. To enable or disable, you can pass the following options, changing out the plugins you want to enable or disable:

``` 
--enable-admission-plugins=NamespaceLifecycle,LimitRanger
--disable-admission-plugins=PodNodeSelector
```

Controllers becoming more common are MutatingAdmissionWebhook and ValidatingAdmissionWebhook, which will allow the dynamic modification of the API request, providing greater flexibility. These calls reference an exterior service, such as OPA, and wait for a return API call. Each admission controller functionality is explained in the documentation. For example, the ResourceQuota controller will ensure that the object created does not violate any of the existing quotas.
</details>


###### Helm
<details>
<summary>Click to expand Helm!</summary>
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

</details>

###### Serverless
<details>
<summary>Click to expand Serverless!</summary>
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
</details>

###### Kubeless
<details>
<summary>Click to expand Kubeless!</summary>
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
</details>

###### Kops (Kubernetes Operations)
<details>
<summary>Click to expand Kops!</summary>
Tool used to spin up a highly available production cluster.
Used to setup Kubernetes on AWS.
Works only on Mac / Linux.
</details>


###### Commands
<details>
<summary>Click to expand Commands!</summary>
Convert dokcer-compse to K8s objects:

``` bash
curl -L https://bit.ly/2tN0bEa -o kompose
kompose convert -f docker-compose.yaml -o localregistry.yaml
```

Run command over couple of nodes:

``` bash
for name in $(kubectl get pod -l app=try1 -o name)> do> kubectl exec $name -c simpleapp -- touch /tmp/healthy> done
```

Debug network with calicoctl
``` bash
curl -O -L https://github.com/projectcalico/calicoctl/releases/download/v3.11.1/calicoctl
sudo calicoctl node status
sudo iptables -I INPUT 5 -i eth0 -p tcp --dport 179 -m state --state NEW,ESTABLISHED -j ACCEPT
```

Evaluate Network Plugins
``` bash
less /var/log/calico/cni/cni.log
```


</details>

