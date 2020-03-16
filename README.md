# aws-eks-notes
Quick notes for AWS EKS

## EKS?
Managed Kubernetes PaaS from AWS. EKS manages the control plane and you get integrations with others services. 

| DIY | Tools  | EKS  |
|:-:|:-:|:-:|
| From Scratch on EC2  | eksctl, kops, kubespray, kubeadm  | EKS + Terraform  |
| Total customization  | Reduced up front work  | Fully managed control plane, AWS integrations, less ops burden  |
|  3-6 months prod grade, 100% burden on team | hard to keep defined in code, all burden on you except eksctl create cluster  |  EKS is new(ish) |

## EKS Managed Control Plane

### Kubernetes API Server
This is the API users interact with for example with `kubectl`

### Scheduler
The scheduler is resposible for picking which worker nodes should run the containers.

### Controller Manager
Runs all the controllers that watch the state of the cluster and when necessary make changes to put the cluster in the desired state. For example the node controller watches number of nodes running and the replication controller watches the number of containers running. 

### etcd
A distributed key-value store the master nodes use to store cluster configuration.

## Worker Nodes - What you manage
Worker nodes run containers and have a few components they run

### Kubelet
The primary agent on each worker nodes. Coordinates with API server and manages the containers it is supposed to be running. 

### kube-proxy
Talks to API server and manages service discovery within the cluster. 

## Kubernetes Access Control

### User Accounts
Accounts used by humans or services OUTSIDE the kubernetes cluster. For example, team members. With EKS the user accounts are likely IAM user accounts who belong to a particular group.

### Service Accounts
Service accounts are managed and used WITHIN the Kubernetes cluster, such as your pods. Kubernetes creates some service accounts automatically; you can create others using the Kubernetes API. The creds for service accounts are stored as secrets in Kubernetes and mounted into pods that should have access.

## K8s Resources
Before we continue it might help to refresh on K8s resources. K8s resources include `pods`, `controllers`, `namespaces`, `services` and `configuration`.

### Pods
The basic building block is a Pod, not a container. A pod can be one or more containers deployed together.
The pod is a logical machine. The containers run in the same namespaces and talk over localhost (different ports).
It is critical to understand for resiliant design that pods are ephermeral. Pods crash and are replaced at any time.

#### Pod Deployment
The scheduler will decide which worker node the pod should deploy on on will deploy the containers for that pod together.

#### Sidecars
It is common to run pods with supporting services called sidecars. For example, proxy (envoy) or log aggregation (fluentd, fluentbit).

### Controllers
A pod runs on a node. A controller is a higher level construct that allows you to run multiple pods for availability or scale accross mulitiple nodes. Most common type is Deployment controller which allows you to specify: What to deploy, how many replicas, and how to roll out updates when you make a change. 

### Namespaces
Namespaces logically partition the clusters into multiple virtual clusters. 

### Services and Discovery
Use a K8s Service to provide a single endpoint in front of a set of pods (which have changing IPs on every deployment). 
Services determine load balancing and route traffic to pods. Typically apps can discover other services by getting the endpoint IP from DNS (via a cluster add-on). 

### Configuration and Secrets
ConfigMaps can be used to provide the configuration for containers in different enviroments (dev,stage,prod). These values can be strings or entire files. ConfigMaps are stored in etcd and exposed to containers as envionment variables or files mounted in a volume. 

To pass secrcets you can use a Kubernetes Secret. The difference is that they are encrypted and only ever store in memory. (If exposing as a file mounted in a volume the secret is mounted in RAM via tmpfs. 

