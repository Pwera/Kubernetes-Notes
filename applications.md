# Kubernetes for application developers:
1. [Pod](#pod)
2. [Service](#service)
3. [Secrets](#secrets)
4. [Service Discovery using DNS](#service-discovery-using-dns)
5. [ConfigMap](#configmap)
6. [Ingress](#ingress)
7. [External DNS](#external-dns)
8. [Egress Gateway](#egress-gateway)
9. [Volumes](#volumes)
10. [Pod Presets](#pod-presets)
11. [StatefulSets](#statefulsets)
12. [DeamonSets](#deamonsets)
13. [Operators](#operators)

## Pod
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


## Service

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



##Scaling pods

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



##Deployments


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



##Services


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


##Labels


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


##Health checks

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


##Readiness Probe.

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


##Pod state

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


## Secrets
 
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


## Service Discovery using DNS
 
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



## ConfigMap

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


## Ingress

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

## External DNS
On public cloud providers, you can use the ingress controller to reduce the cost of LoadBalancer.
You can use one LB that capures all the external traffic and send it to the ingress controller.
The ingress controller can be configured to route different traffic to your apps based on http rules(host and prefixes).
This only works for https- based applications.



## Egress Gateway

Every traffic that needs to exit the service mesh needs to go via Egress Gateway.



## Volumes

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


Volumes AutoProvisioning
The Kubernetes plugins have the capability to provision storage.
This is done using the StorageClass object.


## Pod Presets

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


## StatefulSets

It's introduced to be able to run stateful applications.
A StatefulSet will allow stateful app to use DNS to find other peers.
```bash
kubectl create -f k8s/statefulset/cassandra.yaml
kubectl get pod
kubectl get pv
watch kubectl get pod
```



## DeamonSets

Deamon Sets ensure that every single node in the Kubernetes cluster runs that same pod resource.
This is useful if you want to ensure that a certain pod is running on every single kubernetes node.
When a node is added to the cluster, a new pod will be started automatically.
Same, when a node is removed, the pod will not be rescheduled on another node.




## Operators

Is a method of packaging, deploying and managing a Kubernetes application.
An operator contains a lot of the management logic that you as an administrator or user might want, rather than having to implement it yourself.