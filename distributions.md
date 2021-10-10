# Distributions:
1. [kubadm](#kubadm)
2. [Minikube](#minikube)
3. [Kops (Kubernetes Operations)](#kops-kubernetes-operations))


## kubadm

Installing k8s controll plane:

<table>
<tr>
<td>Install <i>docker</i>, <i>kubeadm</i>,<br><i>kubelet</i>,<br><i>kubectl</i>
<td>

```bash
sudo -i
apt-get update && apt-get upgrade -y
apt-get install -y docker.io

# curl -s \
https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| apt-key add -

apt-get update
apt-get install -y \
kubeadm=1.20.1-00 kubelet=1.20.1-00 kubectl=1.20.1-00
apt-mark hold kubelet kubeadm kubectl
```
<tr>
<td>Install CNI- calico
<td>

```bash
wget https://docs.projectcalico.org/manifests/calico.yaml
```

<tr>
<td>Modify hosts entry
<td>

```bash
hostname -i
ip addr show
vim /etc/hosts
10.128.0.3 k8scp #<-- Add this line
127.0.0.1 localhost
```


<tr>
<td>Run kubeadm
<td>

```bash
vim kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: 1.20.1
controlPlaneEndpoint: "k8scp:6443"
networking:
  podSubnet: 192.168.0.0/16 #<-- Match the IP range from the Calico config file

  # journalctl -xeu kubelet | less
kubeadm init --config=kubeadm-config.yaml --upload-certs \
| tee kubeadm-init.out  
```

<tr>
<td>Post installation bash completion and installation script with token
<td>

```bash
sudo apt-get install bash-completion -y
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> $HOME/.bashrc
sudo kubeadm config print init-defaults
```

</table>

Grow the cluster:

<table>
<tr>
<td>On worker node
<td>

``` bash
sudo -i
apt-get update && apt-get upgrade -y
apt-get install -y docker.io
vim /etc/apt/sources.list.d/kubernetes.list
curl -s \
https://packages.cloud.google.com/apt/doc/apt-key.gpg \
| apt-key add -
apt-get update
 apt-get install -y \
kubeadm=1.21.1-00 kubelet=1.21.1-00 kubectl=1.21.1-00
apt-mark hold kubeadm kubelet kubectl

```

<tr>
<td>On master node
<td>

``` bash
sudo kubeadm token list
# if token was outdated
sudo kubeadm token create
openssl x509 -pubkey \
-in /etc/kubernetes/pki/ca.crt | openssl rsa \
-pubin -outform der 2>/dev/null | openssl dgst \
-sha256 -hex | sed 's/ˆ.* //'



```

<tr>
<td>On worker node
<td>

``` bash
vim /etc/hosts
10.128.0.3 k8scp
#<-- Add this line
127.0.0.1 localhost

kubeadm join \
--token 27eee4.6e66ff60318da929 \
k8scp:6443 \
--discovery-token-ca-cert-hash \
sha256:6d541678b05652e1fa5d43908e75e67376e994c3483d6683f2a18673e5d2aABC
```

</table>


## Minikube

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


## Kops (Kubernetes Operations)

Tool used to spin up a highly available production cluster.
Used to setup Kubernetes on AWS.
Works only on Mac / Linux.