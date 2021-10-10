# Administration:
1. [Autoscaling](#autoscaling)
2. [Affinity, anti-affinity](#affinity-anti-affinity)
3. [Custom Resource Definitions](#custom-resource-definitions)
4. [Namespaces](#namespaces)
5. [Authentication Authorization Admission Control](#authentication-authorization-admission-control)
6. [Node Maintenance](#node-maintenance)
7. [Resource Usage Monitoring (deprecated)](#resource-usage-monitoring-deprecated)

8. [Upgrade controlplane & worker nodes](#upgrade-controlplane-worker-nodes)

9. [Backup The etcd database](#backup-the-etcd-database)
10. [LimitRange - Resource Limits for a Namespace](#limitrange-resource-limits-for-a-namespace)
11. [Configuring TLS Access](#configuring-tls-access)



## Autoscaling

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


## Affinity, anti-affinity

Affinity, anti-affinity feature allows to do more complex scheduling then the nodeSelector and also works on pods.
The rules are not hard requirements, bu rather a prefferred rule.
For example, a rule that makes sure 2 different pods will never be on the same node.


## Custom Resource Definitions

Lets extend the Kubernetes API.
Resources are the endpoints in the Kubernetes API that store collections of API Objects.
For example, there is the built-in Deployment resource, that can be used to deploy applications.
In the yaml files you describe the object, using the Deployment resource type.
You crete the object on the cluster by using kubectl.
Operators, exmplained in the next lecture, use these CRDs to extend the Kubernets API, with their own functionality.


## Namespaces

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

## Authentication Authorization Admission Control

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






## Node Maintenance

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

## Resource Usage Monitoring (deprecated)

Heapster enables Container Cluster Monitoring and Performance Analysis.
It's providing a monitoring platform for Kubernetes.
Heapster exports clusters metricks via REST endpoints.
You can use different backends with Heapster.


## Upgrade controlplane & worker nodes

``` bash
Upgrade controlplane (assuming that current version is v1.19):
0) k drain controlplane --ignore-daemonsets
1) kubeadm upgrade plan
2) apt-get upgrade -y kubeadm=1.20.0-00 --allow-change-held-packages
3) kubeadm upgrade apply v1.20.0
4) kubectl get node (will show kubelet version)
5) apt-get upgrade -y kubelet=1.20.0-00 --allow-change-held-packages
6) systemctl restart kubelet
7) kubectl get node


Worker nodes:
0) apt-get update
1) k drain node01 --ignore-daemonsets
2) apt-get upgrade -y kubeadm=1.20.0-00 --allow-change-held-packages
3) apt-get upgrade -y kubelet=1.20.0-00 --allow-change-held-packages
4) kubeadm upgrade node config --kubelet-version v1.20.0
5) systemctl restart kubelet
6) k uncordon node01
```




## Backup The etcd database

<table>
<tr>
<td>Prepare backup file

<td>

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db
ls snapshot.db
```
<tr>
<td>Backup status
<td>

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

<tr>
<td>Backup restore
<td>

```
service kube-apiserver stop
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db --data-dir /var/lib/etcd-from-backup
```

<tr>
<td>
<td>

```bash
change etcd.service
add --data-dir=/var/lib/etcd-from-backup
sudo systemctl daemon-reload
service etcd restart
service kube-apiserver start
```

<tr>
<td>
<td>

```bash
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db


root@controlplane:~# ETCDCTL_API=3 etcdctl  --data-dir /var/lib/etcd-from-backup \
snapshot restore /opt/snapshot-pre-boot.db
```

```bash
 kubectl -n kube-system exec -it etcd-master2 -- sh -c \
"ETCDCTL_API=3 --cert=./peer.crt --key=./peer.key --cacert=./ca.crt \
etcdctl --endpoints=https://127.0.0.1:2379 member list"
```

</table>


## LimitRange - Resource Limits for a Namespace

<table>
<tr>
<td>Resource Limits for a Namespace
<td>

``` bash
kubectl create namespace low-usage-limit
```

``` yaml

apiVersion: v1
kind: LimitRange
metadata:
  name: low-resource-range
spec:
    limits:
    - default:
        cpu: 1
        memory: 500Mi
    defaultRequest:
        cpu: 0.5
        memory: 100Mi
    type: Container
```
</table>


## Configuring TLS Access

```bash
export client=$(grep client-cert $HOME/.kube/config |cut -d" " -f 6)

export key=$(grep client-key-data $HOME/.kube/config |cut -d " " -f 6)

export auth=$(grep certificate-authority-data $HOME/.kube/config |cut -d " " -f 6)

echo $client | base64 -d - > ./client.pem

echo $key | base64 -d - > ./client-key.pem

echo $auth | base64 -d - > ./ca.pem

curl --cert ./client.pem \
--key ./client-key.pem \
--cacert ./ca.pem \
https://k8scp:6443/api/v1/pods
```