# Networking:
1. [Pod Security Policies](#pod-security-policies)
2. [Admission Controller](#admission-controller)
2. [Service Accounts](#service-accounts)



## Pod Security Policies

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

## Admission Controller
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

## Service Accounts

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