# K8S Bare metal Load Balancer
	k2r2.bai@gmail.com

### K8S is ver flexible to deploy
- k8s is very flexible in how you can deploy it. You can deploy to cloud environments like Google Cloud, Microsoft Azure, and Amazon AWS.
- You can deploy K8s on bare metal using serveral popular operationg systems like Ubuntu Linux, CentOS.

### Limitations of On-Premises
- K8s does not offer an implementation of network load-balancers(Services of type LB) for bare metal(On-Premises) clusters.
- Bare metal cluster operators are left with two lesser tools to bring user traffic into their clusters, "NodePort" and "externalIPs" services.
- Both of these options have significant donsides for productions use, which makes bare metal cluster

### K8s Custom Resources
- A resource is an endpoint in the k8s API that stores a collection of API objects of a certain kind

### API Aggregation
- Require coding, built atop k8s.io/apiserver library
- Highly customizable, like adding a new verb, create/delete hooks.
- Typed fields, validation, defaults.

### K8s Custom Controllers
- K8s 1.7 has added an important feature called Custom Controllers
- It enables developers to extend and add new functionalities, replace existent ones(like replacing kube-proxy for instance)

### Example: PA Firewall + K8s
- Provides Security and NAT custom resources.
- Automatically sync PA security and NAT policies.

### How to create a controllers?
- client-go

### K8s OPerators
- An Operator is nothing more than a set of application-specific custom controllers.
- the Operator monitors and analyzes the cluster, and based on a set of parameters, trigger a series of actions to achieve the desired state.

### What is IPVS?
- IPVS(IP Virtual Server) implements transport-layer load balancing, usually called Layer 4 LAN switching, as part of Linux Kernel.
- IPVS is incorporated into the LVS(Linux Virtual Server), where it runs on a host and acts as a load balancer in front of a cluster of real servers.
- Same to IPTables, IPVS is built on top of Netfilter.
- Support 3 load balancing mode: DNAT, DR(orDSR) and IP tunnel.