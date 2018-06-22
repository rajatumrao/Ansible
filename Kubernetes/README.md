Introduction
Kubernetes is a powerful system, developed by Google, for managing containerized applications in a clustered environment. It aims to provide better ways of managing related, distributed components across varied infrastructure.

In this guide, we'll discuss some of Kubernetes' basic concepts. We will talk about the architecture of the system, the problems it solves, and the model that it uses to handle containerized deployments and scaling.

Prerequisites
This article assumes some prior knowledge about modern cluster technology. We will be discussing how it relates to systems like CoreOS.

If you are not familiar with CoreOS, it may be helpful to review some basic information about the CoreOS system in order to understand the types of environments that Kubernetes is meant to be deployed on.

What is Kubernetes ?
Kubernetes, at its basic level, is a system for managing containerized applications across a cluster of nodes. In many ways, Kubernetes was designed to address the disconnect between the way that modern, clustered infrastructure is designed, and some of the assumptions that most applications and services have about their environments.

Most clustering technologies strive to provide a uniform platform for application deployment. The user should not have to care much about where work is scheduled. The unit of work presented to the user is at the "service" level and can be accomplished by any of the member nodes.

However, in many cases, it does matter what the underlying infrastructure looks like. When scaling an app out, an administrator cares that the various instances of a service are not all being assigned to the same host.

On the other side of things, many distributed applications build with scaling in mind are actually made up of smaller component services. These services must be scheduled on the same host as related components if they are going to be configured in a trivial way. This becomes even more important when they rely on specific networking conditions in order to communicate appropriately.

While it is possible with most clustering software to make these types of scheduling decisions, operating at the level of individual services is not ideal. Applications comprised of different services should still be managed as a single application in most cases. Kubernetes provides a layer over the infrastructure to allow for this type of management.

Master Components
Infrastructure-level systems like CoreOS strive to create a uniform environment where each host is disposable and interchangeable. Kubernetes, on the other hand, operates with a certain level of host specialization.

The controlling services in a Kubernetes cluster are called the master, or control plane, components. These operate as the main management contact points for administrators, and also provide many cluster-wide systems for the relatively dumb worker nodes. These services can be installed on a single machine, or distributed across multiple machines.

The servers running these components have a number of unique services that are used to manage the cluster's workload and direct communications across the system. Below, we will cover these components.

Etcd
One of the fundamental components that Kubernetes needs to function is a globally available configuration store. The etcd project, developed by the CoreOS team, is a lightweight, distributed key-value store that can be distributed across multiple nodes.

Kubernetes uses etcd to store configuration data that can be used by each of the nodes in the cluster. This can be used for service discovery and represents the state of the cluster that each component can reference to configure or reconfigure themselves. By providing a simple HTTP/JSON API, the interface for setting or retrieving values is very straight forward.

Like most other components in the control plane, etcd can be configured on a single master server or, in production scenarios, distributed among a number of machines. The only requirement is that it be network accessible to each of the Kubernetes machines.

API Server
One of the most important master services is an API server. This is the main management point of the entire cluster, as it allows a user to configure many of Kubernetes' workloads and organizational units. It also is responsible for making sure that the etcd store and the service details of deployed containers are in agreement. It acts as the bridge between various components to maintain cluster health and disseminate information and commands.

The API server implements a RESTful interface, which means that many different tools and libraries can readily communicate with it. A client called kubecfg is packaged along with the server-side tools and can be used from a local computer to interact with the Kubernetes cluster.

Controller Manager Service
The controller manager service is a general service that has many responsibilities. It is responsible for a number of controllers that regulate the state of the cluster and perform routine tasks. For instance, the replication controller ensures that the number of replicas defined for a service matches the number currently deployed on the cluster. The details of these operations are written to etcd, where the controller manager watches for changes through the API server.

When a change is seen, the controller reads the new information and implements the procedure that fulfills the desired state. This can involve scaling an application up or down, adjusting endpoints, etc.

Scheduler Service
The process that actually assigns workloads to specific nodes in the cluster is the scheduler. This is used to read in a service's operating requirements, analyze the current infrastructure environment, and place the work on an acceptable node or nodes.

The scheduler is responsible for tracking resource utilization on each host to make sure that workloads are not scheduled in excess of the available resources. The scheduler must know the total resources available on each server, as well as the resources allocated to existing workloads assigned on each server.

Node Server Components
In Kubernetes, servers that perform work are known as nodes. Node servers have a few requirements that are necessary to communicate with the master components, configure the networking for containers, and run the actual workloads assigned to them.

Docker Running on a Dedicated Subnet
The first requirement of each individual node server is docker. The docker service is used to run encapsulated application containers in a relatively isolated but lightweight operating environment. Each unit of work is, at its basic level, implemented as a series containers that must be deployed.

One key assumption that Kubernetes makes is that a dedicated subnet is available to each node server. This is not the case with many standard clustered deployments. For instance, with CoreOS, a separate networking fabric called flannel is needed for this purpose. Docker must be configured to use this so that it can expose ports in the correct fashion.

Kubelet Service
The main contact point for each node with the cluster group is through a small service called kubelet. This service is responsible for relaying information to and from the control plane services, as well as interacting with the etcd store to read configuration details or write new values.

The kubelet service communicates with the master components to receive commands and work. Work is received in the form of a "manifest" which defines the workload and the operating parameters. The kubelet process then assumes responsibility for maintaining the state of the work on the node server.

Proxy Service
In order to deal with individual host subnetting and in order to make services available to external parties, a small proxy service is run on each node server. This process forwards requests to the correct containers, can do primitive load balancing, and is generally responsible for making sure the networking environment is predictable and accessible, but isolated.

Kubernetes Work Units
While containers are the used to deploy applications, the workloads that define each type of work are specific to Kubernetes. We will go over the different types of "work" that can be assigned below.

Pods
A pod is the basic unit that Kubernetes deals with. Containers themselves are not assigned to hosts. Instead, closely related containers are grouped together in a pod. A pod generally represents one or more containers that should be controlled as a single "application".

This association leads all of the involved containers to be scheduled on the same host. They are managed as a unit and they share an environment. This means that they can share volumes and IP space, and can be deployed and scaled as a single application. You can and should generally think of pods as a single virtual computer in order to best conceptualize how the resources and scheduling should work.

The general design of pods usually consists of the main container that satisfies the general purpose of the pod, and optionally some helper containers that facilitate related tasks. These are programs that benefit from being run and managed in their own container, but are heavily tied to the main application.

Horizontal scaling is generally discouraged on the pod level because there are other units more suited for the task.

Services
We have been using the term "service" throughout this guide in a very loose fashion, but Kubernetes actually has a very specific definition for the word when describing work units. A service, when described this way, is a unit that acts as a basic load balancer and ambassador for other containers. A service groups together logical collections of pods that perform the same function to present them as a single entity.

This allows you to deploy a service unit that is aware of all of the backend containers to pass traffic to. External applications only need to worry about a single access point, but benefit from a scalable backend or at least a backend that can be swapped out when necessary. A service's IP address remains stable, abstracting any changes to the pod IP addresses that can happen as nodes die or pods are rescheduled.

Services are an interface to a group of containers so that consumers do not have to worry about anything beyond a single access location. By deploying a service, you easily gain discover-ability and can simplify your container designs.

Replication Controllers
A more complex version of a pod is a replicated pod. These are handled by a type of work unit known as a replication controller.

A replication controller is a framework for defining pods that are meant to be horizontally scaled. The work unit is, in essence, a nested unit. A template is provided, which is basically a complete pod definition. This is wrapped with additional details about the replication work that should be done.

The replication controller is delegated responsibility over maintaining a desired number of copies. This means that if a container temporarily goes down, the replication controller might start up another container. If the first container comes back online, the controller will kill off one of the containers.

Labels
A Kubernetes organizational concept outside of the work-based units is labeling. A label is basically an arbitrary tag that can be placed on the above work units to mark them as a part of a group. These can then be selected for management purposes and action targeting.

Labels are fundamental to how both services and replication controllers function. To get a list of backend servers that a service should pass traffic to, it usually selects containers based on label.

Similarly, replication controllers give all of the containers spawned from their templates the same label. This makes it easy for the controller to monitor each instance. The controller or the administrator can manage all of the instances as a group, regardless of how many containers have been spawned.

Labels are given as key-value pairs. Each unit can have more than one label, but each unit can only have one entry for each key. You can stick with giving pods a "name" key as a general purpose identifier, or you can classify them by various criteria such as development stage, public accessibility, application version, etc.

In many cases, you'll want to assign many labels for fine-grained control. You can then select based on a single or combined label requirements.

Conclusion
Kubernetes is an exciting project that implements many functional improvements on top of clustered infrastructure. While other technologies do a great job at handling the clustering aspects, Kubernetes aims to offer a better management system.

In our next guide, we will discuss how to get Kubernetes up and running on a CoreOS cluster

Ref :- https://www.digitalocean.com/community/tutorials/an-introduction-to-kubernetes
