# certified-k8s-admin
## Table of Contents
 - [Core Concepts](#core-concepts)

## Core Concepts
 - Cluster Architecture
   - Master node:
     - ETCD cluster: stores the information about the cluster
     - kube-scheduler: schedule apps or containers on nodes
     - kube controller manager: take care of different functions, such as node controller, replication controller, etc.
     - kube apiserver: be responsible for orchestrating all operations within the cluster
   - Worker node:
     - kubelet: listen to Kube apiserver for instructions and manage containers
     - kube-proxy: help communicate between services within the cluster
 - Docker vs ContainerD
   - 
##
