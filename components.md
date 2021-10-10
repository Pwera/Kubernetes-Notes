# Components:
1. [etcd](#etcd)
1. [kubelet](#kubelet)
1. [kube-proxy](#kube-proxy)

<i>Kubernetes</i> is and open source orchestration system for containers. It lets you schedule containers on a cluster of machines. Kubernetes can run multiple contianers on one machine. Kubernetes will manage the state of these containers. Can start the contaienr on specific nodes. Will restart container when it gets killed. Can move containers from one node to another node. Containers inside same pod can communicate with localhost.

## etcd







## kubelet

<i>Kubelets</i> are responsible to launch the pods. It's going to connect to the master node to get this information.

## kube-proxy
<i>Kube-proxy</i> is going to feed its information about what pods are on nodes to iptables. Iptables is the firewall in Linux and it can also route traffic.
So whenever a new pod is launched the kube-proxy is going to change the Iptables rules to make sure that the pod is routable within the cluster.

In the iptables proxy mode, kube-proxy continues to monitor the API server for changes in Service and Endpoint objects, and updates rules for each object when created or removed. One limitation to the new mode is an inability to connect to a Pod should the original request fail, so it uses a Readiness Probe to ensure all containers are functional prior to connection. This mode allows for up to approximately 5000 nodes. Assuming multiple Services and Pods per node, this leads to a bottleneck in the kernel.

Another mode beginning in v1.9 is ipvs. While in beta, and expected to change, it works in the kernel space for greater speed, and allows for a configurable load-balancing algorithm, such as round-robin, shortest expected delay, least connection and several others. This can be helpful for large clusters, much past the previous 5000 node limitation. This mode assumes IPVS kernel modules are installed and running prior to kube-proxy. Clusters built with kubeadm do not enable ipvs by default, but this can be passed during cluster initialization.

The kube-proxy mode is configured via a flag sent during initialization, such as mode=iptables and could also be ipvs or userspace.