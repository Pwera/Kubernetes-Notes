# Networking:
1. [Network Policy](#network-policy)

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
Docker uses host-private networking, using the docker0 virtual bridge and veth interfaces would require being on that
host to communicate.


## Network Policy

An early architecture decision with Kubernetes was non-isolation, that all pods were able to connect to all other pods
and nodes by design. In more recent releases the use of a NetworkPolicy allows for pod isolation. The policy only
has effect when the network plugin, like Project Calico, are capable of honoring them. If used with a plugin like flannel
they will have no effect. The use of matchLabels allows for more granular selection within the namespace which
can be selected using a namespaceSelector. Using multiple labels can allow for complex application of rules.


The use of policies has become stable, noted with the v1 apiVersion. The example below narrows down the policy to affect the default namespace.


Only Pods with the label of role: db will be affected by this policy, and the policy has both Ingress and Egress settings.

The ingress setting includes a 172.17 network, with a smaller range of 172.17.1.0 IPs being excluded from this traffic.


``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ingress-egress-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
  - namespaceSelector:
      matchLabels:
        project: myproject
  - podSelector:
      matchLabels:
        role: frontend
  ports:
  - protocol: TCP
    port: 6379
egress:
- to:
  - ipBlock:
      cidr: 10.0.0.0/24
  ports:
  - protocol: TCP
    port: 5978
```

These rules change the namespace for the following settings to be labeled project: myproject. The affected Pods also would need to match the label role: frontend. Finally, TCP traffic on port 6379 would be allowed from these Pods.

The egress rules have the to settings, in this case the 10.0.0.0/24 range TCP traffic to port 5978.

The use of empty ingress or egress rules denies all type of traffic for the included Pods, though this is not suggested. Use another dedicated NetworkPolicy instead.

Some network plugins, such as WeaveNet, may require annotation of the Namespace. The following shows the setting of a DefaultDeny for the myns namespace:


``` yaml
kind: Namespace
apiVersion: v1
metadata:
  name: myns
  annotations:
    net.beta.kubernetes.io/network-policy: |
     {
        "ingress": {
          "isolation": "DefaultDeny"
        }
     }

 ```    

