# Basic Concepts

## Terms

> Node: A machine. (physical or virtual) (formerly: Minion)

> Cluster: A set of nodes.

> Master: A special node on which Kubernetes is installed. It is used to control the other nodes.

## Container

Each component of a software stack lives in a container.

Each container is a separate process, with its own network and memory.

Under each container there is Docker and under it the kernel of the operating system.

Each container is stored in a private or public repository.

It is then retrieved with the command "docker run <NAME_OF_THE_IMAGE>".

> Difference between 'Image' and 'Container': An 'Image' is a software package or template. A 'Container' is an instance of an 'Image'.

## Components

> API Server: A frontend for Kubernetes.

> etcd: A Key-Value-Store for all data needed to manage the cluster, for example all names and addresses of all nodes and masters.

> kubelet: An agent that is installed on each node and monitors whether the containers are running correctly. 

> Container Runtime: The software under Kubernetes for the operation of the containers, in most cases Docker.

> Controller: Registers and reacts when nodes, containers or endpoints fail. 

> Scheduler: Organizes the distribution of containers across multiple nodes.   

## Local and Hosted

A Kubernetes Cluster can be operated locally or on a hosted server.

Local options:
* minikube
* MicroK8s

Hosted options:
* Google Cloud Platform (GCP)
* Amazon Web Services (AWS)
* Microsoft Azure

## Orchestration

A cluster usually operates several instances of a container to ensure load distribution and reliability.

The orchestration of these instances is the main task of Kubernetes.

## Master and Worker Nodes

Master:
* </> kube-apiserver
* etcd
* controller
* scheduler

Worker:
* </> kubelet
* Container Runtime
