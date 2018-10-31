# Chapter 1 Container Orchestration
## what are containers
Containers are an application-centric way to deliver high-performing, scalable applications on the infrastructure of your choice.

> [App	| Bin/lib]	[Desktop, Dev vm,	Qa env,	Public cloud,	Private cloud,	Customer site]

With a container image, we bundle the application along with its runtime and dependencies. We use that image to create an isolated executable environment, also known as container. We can deploy containers from a given image on the platform of our choice, such as desktops, VMs, cloud, etc.

## What Is Container Orchestration?
In the quality assurance (QA) environments, we can get away with running containers on a single host to develop and test applications. However, when we go to production, we do not have the same liberty, as we need to ensure that our applications:

- Are fault-tolerant
- Can scale, and do this on-demand
- Use resources optimally
- Can discover other applications automatically, and communicate with each other
- Are accessible from the external world 
- Can update/rollback without any downtime. 

Container orchestrators are the tools which group hosts together to form a cluster, and help us fulfill the requirements mentioned above.

## Container Orchestrators
Nowadays, there are many container orchestrators available, such as:

- Docker Swarm 

Docker Swarm is a container orchestrator provided by Docker, Inc. It is part of Docker Engine.

- Kubernetes

Kubernetes was started by Google, but now, it is a part of the Cloud Native Computing Foundation project.

- Mesos Marathon

Marathon is one of the frameworks to run containers at scale on Apache Mesos.

- Amazon ECS

Amazon EC2 Container Service (ECS) is a hosted service provided by AWS to run Docker containers at scale on its infrastructrue.

- Hashicorp Nomad

Nomad is the container orchestrator provided by HashiCorp.
We have explored different Container Orchestrators in another edX MOOC, Introduction to Cloud Infrastructure Technologies (LFS151x). We highly recommend that you take LFS151x.

## Why Use Container Orchestrators?
Though we can argue that containers at scale can be maintained manually, or with the help of some scripts, container orchestrators can make things easy for operators.

Container orchestrators can:

- Bring **multiple hosts** together and make them part of a **cluster**
- Schedule containers to run on **different hosts**
- Help containers running on one host **reach out** to containers running on other hosts in the cluster
- Bind containers and storage
- Bind containers of similar type to a higher-level construct, like services, so we don't have to deal with individual containers
- Keep resource usage in-check, and optimize it when necessary
- Allow **secure access** to applications running inside containers.

With all these built-in benefits, it makes sense to use container orchestrators to manage containers. In this course, we will explore Kubernetes. 

## Where to Deploy Container Orchestrators?
Most container orchestrators can be deployed on the infrastructure of our choice. We can deploy them on bare metal, VMs, on-premise, or on a cloud of our choice. For example, Kubernetes can be deployed on our laptop/workstation, inside a company's datacenter, on AWS, on OpenStack, etc. There are even one-click installers available to set up Kubernetes on the cloud, like Google Kubernetes Engine on Google Cloud, or Azure Container Service on Microsoft Azure. Similar solutions are available for other container orchestrators, as well.
There are companies that offer managed Container Orchestration as a Service. We will explore them for Kubernetes in one of the later chapters.

# Chapter 2 Kubernetes

## What Is Kubernetes?

According to the Kubernetes website,
"Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications."
Kubernetes comes from the Greek word κυβερνήτης:, which means helmsman or ship pilot. With this analogy in mind, we can think of Kubernetes as the manager for shipping containers.
Kubernetes is also referred to as k8s, as there are 8 characters between k and s.

Kubernetes is highly inspired by the Google Borg system, which we will explore in this chapter. It is an open source project written in the Go language, and licensed under the Apache License Version 2.0.

Kubernetes was started by Google and, with its v1.0 release in July 2015, Google donated it to the Cloud Native Computing Foundation (CNCF). We will talk more about CNCF later in this chapter.
Generally, Kubernetes has new releases every three months. The current stable version is 1.9 (as of February 2018).

## From Borg to Kubernetes

According to the abstract of Google's Borg paper, published in 2015,
"Google's Borg system is a cluster manager that runs hundreds of thousands of jobs, from many thousands of different applications, across a number of clusters each with up to tens of thousands of machines."
For more than a decade, Borg was Google's secret to run containerized workloads in production. Whatever services we use from Google, like Gmail, Drive, etc., they are all serviced using Borg. 

Some of the initial authors of Kubernetes were Google employees who have used Borg and developed it in the past. They poured in their valuable knowledge and experience while designing Kubernetes. Some of the features/objects of Kubernetes that can be traced back to Borg, or to lessons learnt from it, are:

- API servers
- Pods
- IP-per-Pod
- Services
- Labels.

## Kubernetes Features I

Kubernetes offers a very rich set of features for container orchestration. Some of its fully supported features are:

- Automatic binpacking

Kubernetes automatically schedules the containers based on resource usage and constraints, without sacrificing the availability.

- Self-healing

Kubernetes automatically replaces and reschedules the containers from failed nodes. It also kills and restarts the containers which do not respond to health checks, based on existing rules/policy.

- Horizontal scaling

Kubernetes can automatically scale applications based on resource usage like CPU and memory. In some cases, it also supports dynamic scaling based on customer metrics.

- Service discovery and Load balancing

Kubernetes groups sets of containers and refers to them via a Domain Name System (DNS). This DNS is also called a Kubernetes service. Kubernetes can discover these services automatically, and load-balance requests between containers of a given service.

## Kubernetes Features II

Some other fully supported Kubernetes features are:

- Automated rollouts and rollbacks
Kubernetes can roll out and roll back new versions/configurations of an application, without introducing any downtime.

- Secrets and configuration management
Kubernetes can manage secrets and configuration details for an application without re-building the respective images. With secrets, we can share confidential information to our application without exposing it to the stack configuration, like on GitHub.

- Storage orchestration
With Kubernetes and its plugins, we can automatically mount local, external, and storage solutions to the containers in a seamless manner, based on software-defined storage (SDS).

- Batch execution
Besides long running jobs, Kubernetes also supports batch execution.
There are many other features besides the ones we just mentioned, and they are currently in alpha/beta phase. They will add great value to any Kubernetes deployment once they become stable features. For example, support for role-based access control (RBAC) is  stable as of the Kubernetes 1.8 release.


## Why Use Kubernetes?
We just looked at some of the fully-supported Kubernetes features. We should also mention that Kubernetes is very portable and extensible. Kubernetes can be deployed on the environment of our choice, be it VMs, bare metal, or public/private/hybrid/multi-cloud setups. Also, Kubernetes has a very modular and pluggable architecture. We can write custom APIs or plugins to extend its functionalities.

For a successful open source project, the community is as important as having great code. Kubernetes has a very thriving community across the world. It has more than 1600 contributors, who, over time, have done over 62,000 commits. There are meet-up groups in different cities which meet regularly to discuss Kubernetes and its ecosystem. There are Special Interest Groups (SIGs), which focus on special interests, such as scaling, bare metal, networking, etc. We will talk more about them in our last chapter, Kubernetes Communities.

## Kubernetes Users

With just a few years since its debut, many companies are running workloads using Kubernetes. We can find numerous user case studies on the Kubernetes website:

• Pearson
• Box
• eBay
• Wikimedia
• Huawei
• Haufe Group
• BlackRock
• BlaBlaCar
• And many more.

## Cloud Native Computing Foundation (CNCF)
The Cloud Native Computing Foundation (CNCF) is one of the projects hosted by The Linux Foundation. CNCF aims to accelerate the adoption of containers, microservices, and cloud-native applications.

CNCF hosts a set of projects, with more to be added in the future. CNCF provides resources to each of the projects, but, at the same time, each project continues to operate independently under its pre-existing governance structure and with its existing maintainers. At the time this course was created, the following projects were part of CNCF:

• containerd for container runtime
• rkt for container runtime
• Kubernetes for container orchestration
• Linkerd for service mesh
• Envoy for service mesh
• gRPC for remote procedure call (RPC)
• Container Network Interface (CNI) for networking API
• CoreDNS for service discovery
• Rook for cloud-native storage
• Notary for security
• The Update Framework (TUF) for software updates
• Prometheus for monitoring
• OpenTracing for tracing
• Jaeger for distributed tracing
• Fluentd for logging
• Vitess for storage.
As we can see, this set of CNCF projects can cover the entire lifecycle of an application, from its execution using container runtimes, to its monitoring and logging. This is very important to meet the CNCF goal. 

## CNCF and Kubernetes

For Kubernetes, the Cloud Native Computing Foundation:
• Provides a neutral home for the Kubernetes trademark and enforces proper usage
• Provides license scanning of core and vendored code
• Offers legal guidance on patent and copyright issues
• Creates open source curriculum, training, and certification
• Manages a software conformance working group
• Actively markets Kubernetes
• Hosts and funds developer marketing activities like K8Sport
• Supports ad hoc activities
• Funds conferences and meetup events.


# Chapter 3. Kubernetes Architecture

## Kubernetes Architecture

At a very high level, Kubernetes has the following main components:

- One or more master nodes
- One or more worker nodes
- Distributed key-value store, like etcd.

Next, we will explore the Kubernetes architecture in more detail.

## Master Node
The master node is responsible for managing the Kubernetes cluster, and it is the entry point for all administrative tasks. We can communicate to the master node via the CLI, the GUI (Dashboard), or via APIs.

## Kubernetes Master Node
 
For fault tolerance purposes, there can be more than one master node in the cluster. If we have more than one master node, they would be in a HA (High Availability) mode, and only one of them will be the leader, performing all the operations. The rest of the master nodes would be followers.

To manage the cluster state, Kubernetes uses etcd, and all master nodes connect to it. etcd is a distributed key-value store, which we will discuss in a little bit. The key-value store can be part of the master node. It can also be configured externally, in which case, the master nodes would connect to it.

## Master Node Components

A master node has the following components:

- API server
- Scheduler
- Controller manager
- etcd.

## Master Node Components: API Server

All the administrative tasks are performed via the API server within the master node. A user/operator sends REST commands to the API server, which then validates and processes the requests. After executing the requests, the resulting state of the cluster is stored in the distributed key-value store.

## Master Node Components: Scheduler

As the name suggests, the scheduler schedules the work to different worker nodes. The scheduler has the resource usage information for each worker node. It also knows about the constraints that users/operators may have set, such as scheduling work on a node that has the label disk==ssd set. Before scheduling the work, the scheduler also takes into account the quality of the service requirements, data locality, affinity, anti-affinity, etc. The scheduler schedules the work in terms of Pods and Services.

## Master Node Components: Controller Manager

The controller manager manages different non-terminating control loops, which regulate the state of the Kubernetes cluster. Each one of these control loops knows about the desired state of the objects it manages, and watches their current state through the API server. In a control loop, if the current state of the objects it manages does not meet the desired state, then the control loop takes corrective steps to make sure that the current state is the same as the desired state.

## Master Node Components: etcd
etcd is a distributed key-value store which is used to store the cluster state. It can be part of the Kubernetes Master, or, it can be configured externally, in which case, master nodes would connect to it.

## Worker Node
A worker node is a machine (VM, physical server, etc.) which runs the applications using Pods and is controlled by the master node. Pods are scheduled on the worker nodes, which have the necessary tools to run and connect them. A Pod is the scheduling unit in Kubernetes. It is a logical collection of one or more containers which are always scheduled together. We will explore them further in later chapters.

Also, to access the applications from the external world, we connect to worker nodes and not to the master node/s. We will dive deeper into this in future chapters. 

## Worker Node Components
- Container runtime
- kubelet
- kube-proxy.

## Worker Node Components: Container Runtime
To run and manage a container's lifecycle, we need a container runtime on the worker node. Some examples of container runtimes are: 

- containerd
- rkt
- lxd. 

Sometimes, Docker is also referred to as a container runtime, but to be precise, Docker is a platform which uses containerd as a container runtime. 

## Worker Node Components: kubelet

The kubelet is an agent which runs on each worker node and communicates with the master node. It receives the Pod definition via various means (primarily, through the API server), and runs the containers associated with the Pod. It also makes sure that the containers which are part of the Pods are healthy at all times.

The kubelet connects to the container runtime using **Container Runtime Interface (CRI)**. The Container Runtime Interface consists of protocol buffers, gRPC API, and libraries. 
 

As shown above, the kubelet (grpc client) connects to the CRI shim (grpc server) to perform container and image operations. CRI implements two services: ImageService and RuntimeService. The ImageService is responsible for all the image-related operations, while the RuntimeService is responsible for all the Pod and container-related operations.

Container runtimes used to be hard-coded in Kubernetes, but with the development of CRI, Kubernetes can now use different container runtimes without the need to recompile. Any container runtime that implements CRI can be used by Kubernetes to manage Pods, containers, and container images.

## Worker Node Components: kubelet: CRI shims

Below you will find some examples of CRI shims:

- dockershim
With dockershim, containers are created using Docker installed on the worker nodes. Internally, Docker uses containerd to create and manage containers.

 

- cri-containerd
With cri-containerd, we can directly use Docker's smaller offspring containerd to create and manage containers.

 

- CRI-O
CRI-O enables using any Open Container Initiative (OCI) compatible runtimes with Kubernetes. At the time this course was created, CRI-O supported runC and Clear Containers as container runtimes. However, in principle, any OCI-compliant runtime can be plugged-in.

## Worker Node Components: kube-proxy

Instead of connecting directly to Pods to access the applications, we use a logical construct called a Service as a connection endpoint. A Service groups related Pods and, when accessed, load balances to them. We will talk more about Services in later chapters.

kube-proxy is the network proxy which runs on each worker node and listens to the API server for each Service endpoint creation/deletion. For each Service endpoint, kube-proxy sets up the routes so that it can reach to it. We will also explore this in more detail in later chapters.

## State Management with etcd

As we mentioned earlier, Kubernetes uses etcd to store the cluster state. etcd is a distributed key-value store based on the Raft Consensus Algorithm. Raft allows a collection of machines to work as a coherent group that can survive the failures of some of its members. At any given time, one of the nodes in the group will be the master, and the rest of them will be the followers. Any node can be treated as a master.

etcd is written in the Go programming language. In Kubernetes, besides storing the cluster state, etcd is also used to store configuration details such as subnets, ConfigMaps, Secrets, etc. 

## Network Setup Challenges

To have a fully functional Kubernetes cluster, we need to make sure of the following:

- A unique IP is assigned to each Pod
- Containers in a Pod can communicate to each other
- The Pod is able to communicate with other Pods in the cluster
- If configured, the application deployed inside a Pod is accessible from the external world.

All of the above are networking challenges which must be addressed before deploying the Kubernetes cluster. Next, we will see how we can solve these challenges.

## Assigning a Unique IP Address to Each Pod

In Kubernetes, each Pod gets a unique IP address. For container networking, there are two primary specifications:

Container Network Model (CNM), proposed by Docker
Container Network Interface (CNI), proposed by CoreOS.
Kubernetes uses CNI to assign the IP address to each Pod.

The container runtime offloads the IP assignment to CNI, which connects to the underlying configured plugin, like Bridge or MACvlan, to get the IP address. Once the IP address is given by the respective plugin, CNI forwards it back to the requested container runtime.

## Container-to-Container Communication Inside a Pod

With the help of the underlying host operating system, all of the container runtimes generally create an isolated network entity for each container that it starts. On Linux, that entity is referred to as a network namespace. These network namespaces can be shared across containers, or with the host operating system.

Inside a Pod, containers share the network namespaces, so that they can reach to each other via localhost.

## Pod-to-Pod Communication Across Nodes

In a clustered environment, the Pods can be scheduled on any node. We need to make sure that the Pods can communicate across the nodes, and all the nodes should be able to reach any Pod. Kubernetes also puts a condition that there shouldn't be any Network Address Translation (NAT) while doing the Pod-to-Pod communication across hosts. We can achieve this via:

Routable Pods and nodes, using the underlying physical infrastructure, like Google Kubernetes Engine
Using Software Defined Networking, like Flannel, Weave, Calico, etc. 
For more details, you can take a look at the available Kubernetes documentation.

## Communication Between the External World and Pods
By exposing our services to the external world with kube-proxy, we can access our applications from outside the cluster. We will have a complete chapter dedicated to this, so we will dive into this later.

#Chapter 4. Installing Kubernetes
#Kubernetes Configuration

Kubernetes can be installed using different configurations. The four major installation types are briefly presented below:

- All-in-One Single-Node Installation

With all-in-one, all the master and worker components are installed on a single node. This is very useful for learning, development, and testing. This type should not be used in production. Minikube is one such example, and we are going to explore it in future chapters.

- Single-Node etcd, Single-Master, and Multi-Worker Installation

In this setup, we have a single master node, which also runs a single-node etcd instance. Multiple worker nodes are connected to the master node.

- Single-Node etcd, Multi-Master, and Multi-Worker Installation

In this setup, we have multiple master nodes, which work in an HA mode, but we have a single-node etcd instance. Multiple worker nodes are connected to the master nodes.

- Multi-Node etcd, Multi-Master, and Multi-Worker Installation

In this mode, etcd is configured in a clustered mode, outside the Kubernetes cluster, and the nodes connect to it. The master nodes are all configured in an HA mode, connecting to multiple worker nodes. This is the most advanced and recommended production setup.

## Infrastructure for Kubernetes Installation
Once we decide on the installation type, we also need to make some infrastructure-related decisions, such as:

- Should we set up Kubernetes on bare metal, public cloud, or private cloud?
Which underlying system should we use? 
- Should we choose RHEL, CoreOS, CentOS, or something else?
- Which networking solution should we use?

And so on.

## Localhost Installation

There are a few localhost installation options available to deploy single- or multi-node Kubernetes clusters on our workstation/laptop:

- Minikube
- Ubuntu on LXD.

Minikube is the preferred and recommended way to create an all-in-one Kubernetes setup. We will be using it extensively in this course.

## On-Premise Installation
Kubernetes can be installed on-premise on VMs and bare metal.

- On-Premise VMs

Kubernetes can be installed on VMs created via Vagrant, VMware vSphere, KVM, etc. There are different tools available to automate the installation, like Ansible or kubeadm.

- On-Premise Bare Metal

Kubernetes can be installed on on-premise bare metal, on top of different operating systems, like RHEL, CoreOS, CentOS, Fedora, Ubuntu, etc. Most of the tools used to install VMs can be used with bare metal as well. 

## Cloud Installation

Kubernetes can be installed and managed on almost any cloud environment:

### Hosted Solutions

With hosted solutions, any given software is completely managed by the provider. The user will just need to pay hosting and management charges. Some examples of vendors providing hosted solutions for Kubernetes are listed below:

- Google Kubernetes Engine (GKE)
- Azure Container Service (AKS)
- Amazon Elastic Container Service for Kubernetes (EKS) - Currently in Tech Preview
- OpenShift Dedicated
- Platform9
- IBM Cloud Container Service.

### Turnkey Cloud Solutions
For Kubernetes, we have some Turnkey Cloud Solutions, with which Kubernetes can be installed with just a few commands on an underlying IaaS platform, such as:

- Google Compute Engine
- Amazon AWS
- Microsoft Azure
- Tectonic by CoreOS.

### Bare Metal
Kubernetes can be installed on bare metal provided by different cloud providers.

# Kubernetes Installation Tools/Resources

While discussing installation configuration and the underlying infrastructure, let's take a look at some useful tools/resources available:

- kubeadm

kubeadm is a first-class citizen on the Kubernetes ecosystem. It is a secure and recommended way to bootstrap the Kubernetes cluster. It has a set of building blocks to setup the cluster, but it is easily extendable to add more functionality. Please note that kubeadm does not support the provisioning of machines.

- KubeSpray

With KubeSpray (formerly known as Kargo), we can install Highly Available Kubernetes clusters on AWS, GCE, Azure, OpenStack, or bare metal. KubeSpray is based on Ansible, and is available on most Linux distributions. It is a Kubernetes Incubator project.

- Kops

With Kops, we can create, destroy, upgrade, and maintain production-grade, highly-available Kubernetes clusters from the command line. It can provision the machines as well. Currently, AWS is officially supported. Support for GCE and VMware vSphere are in alpha stage, and other platforms are planned for the future.
If the existing solutions and tools do not fit your requirements, then you can always install Kubernetes from scratch.

It is worth checking out the Kubernetes The Hard Way GitHub project by Kelsey Hightower, which shares the manual steps involved in bootstrapping a Kubernetes cluster.

#Chapter 5. Setting Up a Single-Node Kubernetes Cluster with Minikube  
## Requirements for Running Minikube

In most of the cases, Minikube runs inside a VM on Linux, Mac, or Windows. Therefore, we need to make sure that we have the supported hardware and the hypervisor to create VMs. Next, we outline the requirements to run Minikube on our workstation/laptop:

- kubectl

kubectl is a binaryused  to access any Kubernetes cluster. Generally, it is installed before starting Minikube, but we can install it later, as well. If kubectl is not found while installing Minikube, we will get a warning message, which can be safely ignored (just remember that we will have to install kubectl later). We will explore kubectl in future chapters.

- On Linux
VirtualBox or KVM hypervisors

NOTE: Minikube also supports a --vm-driver=none  option that runs the Kubernetes components on the host and not in a VM. Docker is required to use this driver, but no hypervisor. If you use --vm-driver=none, be sure to specify a bridge network for Docker. Otherwise, it might change between network restarts, causing loss of connectivity to your cluster.

- On macOS

Hyperkit driver, xhyve driver, VirtualBox or VMware Fusion hypervisors

- On Windows

VirtualBox or Hyper-V hypervisors

- VT-x/AMD-v virtualization must be enabled in BIOS

- Internet connection on first run.

In this chapter, we will use VirtualBox as hypervisor on all three operating systems - Linux, macOS, and Windows, to create the Minikube VM. 


## Installing Minikube on macOS

On macOS, Minikube uses VirtualBox as the default hypervisor, which we will use as well. But, we can also use the xhyve hypervisor, or the hyperkit driver, as well.  We can pass the  `-vm-driver=<driver> ` option while starting the Minikube to use the specific driver. 

Next, we will learn how to install Minikube on macOS:

Install VirtualBox on macOS

Install Minikube 

We can download the latest release from the Minikube release page. Once downloaded, we need to make it executable and copy it in the PATH.

Start Minikube

We can start Minikube with the minikube start command:

Check the status

We can see the status of Minikube with the minikube status command:

Stop Minikube

We can stop Minikube with the minikube stop command:

Minikube CRI-O

According to the CRI-O website,

"CRI-O is an implementation of the Kubernetes CRI (Container Runtime Interface) to enable using OCI (Open Container Initiative) compatible runtimes."

In order to start Minikube with CRI-O, run the following command:

Now, let's do ssh login to the Minikube's VM:

## Installing Minikube Demo

# Chapter 6. Accessing Minikube
## Accessing Minikube

Any healthy running Kubernetes cluster can be accessed via one of the following methods:

- Command Line Interface (CLI)
- Graphical User Interface (GUI)
- APIs.

These methods are applicable to all Kubernetes clusters. 

## Accessing Minikube: Command Line Interface (CLI)

kubectl is the Command Line Interface (CLI) tool to manage the Kubernetes cluster resources and applications. In later chapters, we will be using kubectl to deploy the applications and manage the Kubernetes resources.

## Accessing Minikube: Graphical User Interface (GUI)

The Kubernetes dashboard provides the Graphical User Interface (GUI) to interact with its resources and containerized applications. In one of the later chapters, we will be using it to deploy a containerized application.

## Accessing Minikube: APIs

As we know, Kubernetes has the API server, and operators/users connect to it from the external world to interact with the cluster. Using both CLI and GUI, we can connect to the API server on the master node to perform different operations. We can directly connect to the API server using its API endpoints and send commands to it, as long as we can access the master node and have the right credentials.

Below, you can find a part of the HTTP API space of Kubernetes:

HTTP API space of Kubernetes can be divided into three independent groups:

- Core Group (/api/v1)
This group includes objects such as Pods, Services, nodes, etc.

- Named Group

This group includes objects in /apis/$NAME/$VERSION format. These different API versions imply different levels of stability and support:
Alpha level - it may be dropped at any point in time, without notice. For example, /apis/batch/v2alpha1.
Beta level - it is well-tested, but the semantics of objects may change in incompatible ways in a subsequent beta or stable release. For example, /apis/certificates.k8s.io/v1beta1.
Stable level - appears in released software for many subsequent versions. For example, /apis/networking.k8s.io/v1.

- System-wide

This group consists of system-wide API endpoints, like /healthz, /logs, /metrics, /ui, etc.

We can either connect to an API server directly via calling the respective API endpoints, or via the CLI/GUI.

Next, we will see how we can access the Minikube environment we set up in the previous chapter.

## kubectl

kubectl is generally installed before installing Minikube, but we can also install it later. There are different methods that can be used to install kubectl, which are mentioned in the Kubernetes documentation. Next, we will look at the steps to install kubectl on Linux, macOS, and Windows systems.

## Installing kubectl on macOS
There are two ways to install kubectl on macOS: manually and using the Homebrew package manager. Next, we will provide instructions for both methods.

To manually install kubectl on macOS, follow the instructions below:

Download the latest stable kubectl binary

Make the kubectl binary executable

Move the kubectl binary to the PATH

To install kubectl on macOS using the Homebrew package manager:
Use the following command

## kubectl Configuration File
To connect to the Kubernetes cluster, kubectl needs the master node endpoint and the credentials to connect to it. While starting Minikube, the startup process creates, by default, a configuration file, config, inside the .kube directory, which resides in the user's home directory. That configuration file has all the connection details. By default, the kubectl binary accesses this file to find the master node's connection endpoint, along with the credentials. To look at the connection details, we can either see the content of the ~/.kube/config(Linux) file, or run the following command:
```
kubectl config view
```
Once kubectl is installed, we can get information about the Minikube cluster with the kubectl cluster-info command: 
```kubectl cluster-info```

## Using the 'minikube dashboard' Command

As mentioned earlier, the Kubernetes dashboard provides the user interface for the Kubernetes cluster. To access the dashboard of Minikube, we can use minikube dashboard, which would open a new tab on our web browser, displaying the Kubernetes dashboard:

`$ minikube dashboard`

## Using the 'kubectl proxy' Command
Using the `kubectl proxy` command, kubectl would authenticate with the API server on the master node and would make the dashboard available on ...

## APIs - with 'kubectl proxy'
When kubectl proxy is configured, we can send requests to localhost on the proxy port:
```
$ curl http://localhost:8001/
{
 "paths": [
   "/api",
   "/api/v1",
   "/apis",
   "/apis/apps",
   ......
   ......
   "/logs",
   "/metrics",
   "/swaggerapi/",
   "/ui/",
   "/version"
 ]
}%
```
With the above curl request, we requested all the API endpoints from the API server.

## APIs - without 'kubectl proxy'
Without kubectl proxy configured, we can get the Bearer Token using kubectl, and then send it with the API request. A Bearer Token is an access token which is generated by the authentication server (the API server on the master node) and given back to the client. Using that token, the client can connect back to the Kubernetes API server without providing further authentication details, and then, access resources.

#Chapter 7. Kubernetes Building Blocks  
## Kubernetes Object Model

Kubernetes has a very rich object model, with which it represents different persistent entities in the Kubernetes cluster. Those entities describe:

- What containerized applications we are running and on which node
- Application resource consumption
- Different policies attached to applications, like restart/upgrade policies, fault tolerance, etc.

With each object, we declare our intent or desired state using the spec field. The Kubernetes system manages the status field for objects, in which it records the actual state of the object. At any given point in time, the Kubernetes Control Plane tries to match the object's actual state to the object's desired state.

Examples of Kubernetes objects are Pods, ReplicaSets, Deployments, Namespaces, etc. We will explore them next.

To create an object, we need to provide the spec field to the Kubernetes API server. The spec field describes the desired state, along with some basic information, like the name. The API request to create the object must have the spec field, as well as other details, in a JSON format. Most often, we provide an object's definition in a.yaml file, which is converted by kubectl in a JSON payload and sent to the API server.

Below is an example of a Deployment object:
```
apiVersion: apps/v1

kind: Deployment

metadata:

  name: nginx-deployment

  labels:

    app: nginx

spec:

  replicas: 3

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

        image: nginx:1.7.9

        ports:

        - containerPort: 80
```
With the apiVersion field in the example above, we mention the API endpoint on the API server which we want to connect to. With the kind field, we mention the object type - in our case, we have Deployment. With the metadata field, we attach the basic information to objects, like the name. You may have noticed that in our example we have two spec fields (spec and spec.template.spec). With spec, we define the desired state of the deployment. In our example, we want to make sure that, at any point in time, at least 3 Pods are running, which are created using the Pods Template defined in spec.template. In spec.template.spec, we define the desired state of the Pod. Here, our Pod would be created using nginx:1.7.9.

Once the object is created, the Kubernetes system attaches the status field to the object; we will explore it later.

Next, we will take a closer look at some of the Kubernetes objects, along with other building blocks.

## Pods

A Pod is the smallest and simplest Kubernetes object. It is the unit of deployment in Kubernetes, which represents a single instance of the application. A Pod is a logical collection of one or more containers, which:

- Are scheduled together on the same host
- Share the same network namespace
- Mount the same external storage (volumes).
 
A pod is a collection of one or more containers

Pods are ephemeral in nature, and they do not have the capability to self-heal by themselves. That is why we use them with controllers, which can handle a Pod's replication, fault tolerance, self-heal, etc. Examples of controllers are Deployments, ReplicaSets, ReplicationControllers, etc. We attach the Pod's specification to other objects using Pods Templates, as we have seen in the previous section.

## Labels
Labels are key-value pairs that can be attached to any Kubernetes objects (e.g. Pods). Labels are used to organize and select a subset of objects, based on the requirements in place. Many objects can have the same Label(s). Labels do not provide uniqueness to objects. 

In the image above, we have used two Labels: app and env. Based on our requirements, we have given different values to our four Pods.

## Label Selectors
With Label Selectors, we can select a subset of objects. Kubernetes supports two types of Selectors:

Equality-Based Selectors
Equality-Based Selectors allow filtering of objects based on Label keys and values. With this type of selectors, we can use the =, ==, or != operators. For example, with env==dev we are selecting the objects where the env Label is set to dev. 
Set-Based Selectors
Set-Based Selectors allow filtering of objects based on a set of values. With this type of Selectors, we can use the in, notin, and exist operators. For example, with env in (dev,qa), we are selecting objects where the env Label is set to dev or qa.

## ReplicationControllers

A ReplicationController (rc) is a controller that is part of the master node's controller manager. It makes sure the specified number of replicas for a Pod is running at any given point in time. If there are more Pods than the desired count, the ReplicationController would kill the extra Pods, and, if there are less Pods, then the ReplicationController would create more Pods to match the desired count. Generally, we don't deploy a Pod independently, as it would not be able to re-start itself, if something goes wrong. We always use controllers like ReplicationController to create and manage Pods. 

## ReplicaSets I

A ReplicaSet (rs) is the next-generation ReplicationController. ReplicaSets support both equality- and set-based selectors, whereas ReplicationControllers only support equality-based Selectors. Currently, this is the only difference.

Next, you can see a graphical representation of a ReplicaSet, where we have set the replica count to 3 for a Pod.

## ReplicaSets II

Now, let's suppose that one Pod dies, and our current state is not matching the desired state anymore.

## ReplicaSets III
The ReplicaSet will detect that the current state is no longer matching the desired state. So, in our given scenario, the ReplicaSet will create one more Pod, thus ensuring that the current state matches the desired state.

ReplicaSet (Creating a Pod to Match Current and Desired State)

ReplicaSets can be used independently, but they are mostly used by Deployments to orchestrate the Pod creation, deletion, and updates. A Deployment automatically creates the ReplicaSets, and we do not have to worry about managing them. 

## Deployments I

Deployment objects provide declarative updates to Pods and ReplicaSets. The DeploymentController is part of the master node's controller manager, and it makes sure that the current state always matches the desired state.

In the following example, we have a Deployment which creates a ReplicaSet A. ReplicaSet A then creates 3 Pods. In each Pod, one of the containers uses the nginx:1.7.9 image.
Deployment (ReplicaSet A Created)

## Deployments II
Now, in the Deployment, we change the Pods Template and we update the image for the nginx container from nginx:1.7.9 to nginx:1.9.1. As have modified the Pods Template, a new ReplicaSet B gets created. This process is referred to as a Deployment rollout.

A rollout is only triggered when we update the Pods Template for a deployment. Operations like scaling the deployment do not trigger the deployment.

## Deployments III

Once ReplicaSet B is ready, the Deployment starts pointing to it.

Deployment Points to ReplicaSet B

On top of ReplicaSets, Deployments provide features like Deployment recording, with which, if something goes wrong, we can rollback to a previously known state.

## Namespaces

If we have numerous users whom we would like to organize into teams/projects, we can partition the Kubernetes cluster into sub-clusters using Namespaces. The names of the resources/objects created inside a Namespace are unique, but not across Namespaces.

To list all the Namespaces, we can run the following command:

$ kubectl get namespaces
NAME          STATUS       AGE
default       Active       11h
kube-public   Active       11h
kube-system   Active       11h

Generally, Kubernetes creates two default Namespaces: kube-system and default. The kube-system Namespace contains the objects created by the Kubernetes system. The default Namespace contains the objects which belong to any other Namespace. By default, we connect to the default Namespace. kube-public is a special Namespace, which is readable by all users and used for special purposes, like bootstrapping a cluster. 

Using Resource Quotas, we can divide the cluster resources within Namespaces. We will briefly cover resource quotas in one of the future chapters.

#Chapter 8. Authentication, Authorization, Admission Control

## Authentication, Authorization, and Admission Control - Overview

To access and manage any resources/objects in the Kubernetes cluster, we need to access a specific API endpoint on the API server. Each access request goes through the following three stages:

- Authentication
Logs in a user.
- Authorization
Authorizes the API requests added by the logged-in user.
- Admission Control
Software modules that can modify or reject the requests based on some additional checks, like Quota.

##Authentication I

Kubernetes does not have an object called user, nor does it store usernames or other related details in its object store. However, even without that, Kubernetes can use usernames for access control and request logging, which we will explore in this chapter.

Kubernetes has two kinds of users:

Normal Users
They are managed outside of the Kubernetes cluster via independent services like User/Client Certificates, a file listing usernames/passwords, Google accounts, etc.
Service Accounts
With Service Account users, in-cluster processes communicate with the API server to perform different operations. Most of the Service Account users are created automatically via the API server, but they can also be created manually. The Service Account users are tied to a given Namespace and mount the respective credentials to communicate with the API server as Secrets.
If properly configured, Kubernetes can also support anonymous requests, along with requests from Normal Users and Service Accounts.

































