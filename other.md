# Other:
1. [Commands](#commands)
2. [Helm](#helm)
3. [Serverless](#serverless)
4. [Kubeless](#kubeless)
5. [Container Runtime Interface](#container-runtime-interface)
6. [Container Storage Interface](#container-storage-interface)





## Commands

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

Move workloads to other node, and cordon the node
```bash
kubectl drain node-1
```

Make node not schedulable, does not move pods

```bash
kubectl cordon node-1 
```

Eviction timeout:
Optiopn of kube-controller-manager, by default after 5m kubelet, will terminate pod on node.

Scan ports:
```bash
alias rustscan='docker run -it --rm --name rustscan rustscan/rustscan'
```

###### Recording changes in Deployments

``` bash

kubectl set image deployment ghost --image=ghost:0.9 --record
kubectl get deployments ghost -o yaml
metadata:
    annotations:
            deployment.kubernetes.io/revision: "1"
            kubernetes.io/change-cause: kubectl set image deployment ghost --image=ghost:0.9 --record
```

###### Run container repository

``` yaml
registry:
  image: registry:2
  ports:
    - 127.0.0.1:5000:5000
  environment:
    REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
  volumes:
    - /localdocker/data:/data
```


###### Explore API Calls
One way to view what a command does on your behalf is to use strace

``` bash
sudo apt-get install -y strace
strace kubectl get endpoints
cd /home/$USER/.kube/cache/discovery/

```

## Helm

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



## Serverless

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


## Kubeless

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










## Container Runtime Interface

The goal of the Container Runtime Interface (CRI) is to allow easy integration of container runtimes with kubelet. By providing a protobuf method for API, specifications and libraries, new runtimes can easily be integrated without needing deep understanding of kubelet internals.

The project is in early stage, with lots of development in action. Now that Docker-CRI integration is done, new runtimes should be easily added and swapped out. At the moment, CRI-O, rktlet and frakti are listed as work-in-progress.


## Container Storage Interface

Adoption of Container Storage Interface (CSI) enables the goal of an industry standard interface for container orchestration to allow access to arbitrary storage systems. Currently, volume plugins are "in-tree", meaning they are compiled and built with the core Kubernetes binaries. This "out-of-tree" object will allow storage vendors to develop a single driver and allow the plugin to be containerized. This will replace the existing Flex plugin which requires elevated access to the host node, a large security concern.



