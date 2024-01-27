# certified-k8s-admin
## Table of Contents
 - [Certification Tip](#certification-tip)
 - [Core Concepts](#core-concepts)

## Certification Tip
As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. Using the `kubectl run` command can help in generating a YAML template. And sometimes, you can even get away with just the `kubectl run` or `kubectl create` command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the `kubectl run` or `kubectl create` command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises

Create an NGINX Pod

`kubectl run nginx --image=nginx`

Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)

`kubectl run nginx --image=nginx --dry-run=client -o yaml`

Create a deployment

`kubectl create deployment --image=nginx nginx`

Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.

`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

Make necessary changes to the file (for example, adding more replicas) and then create the deployment.

`kubectl create -f nginx-deployment.yaml`

OR

In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.

`kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

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
 - ETCD
 - Kube-API Server
 - Kube Controller Manager
 - Kube Scheduler
 - Kubelet
 - Kube Proxy
 - Pods
 - ReplicaSets: can control pods across multiple nodes
   - commands:
     - `kubectl create -f <replicaset yml file name>.yml`
     - `kubectl get replicaset`
     - `kubectl delete replicaset <replicaset yml file name>`
     - `kubectl replace -f <replicaset yml file name>.yml`
     - `kubectl scale --replicas=6 -f <replicaset yml file name>.yml` --> (-f filename | type name)
 - Deployments: compared to replicaSet, deployment has more advanced features, like `rolling update`, `roll back`, `make changes to the underlying env (pause and resume)`.
   - command: `kubectl get all` 
 - Services:
   - ports: targetPort(if not provided, then it assumes the same as `Port`), Port, NodePort(if not provided, then a valid random port(30000-32767) will be assigned)
   - types: NodePort, ClusterIP(default, can be omitted), LoadBalancer
   - `can be used for cross-node`
 - Namespaces: analogy --> folders with files(resources) /  isolation & resources limit
   - access resources:
     - in the same namespace: `mysql.connect("db-service")`
     - in another namespace: `mysql.connect("db-service.<namespace>.svc.cluster.local")`
   - commands:
     - `kubectl config set-context $(kubectl config current-context) --namespace=<another namespace>`  --> switch namespace
     - add `--namespace=` for some commands to make sure operations performed in the designated namespace
     - `--all-namespaces` tag to apply to all namespaces
     - create `ResourceQuota` for different namespaces
 - Imperative & Declarative:
   - `--dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `--dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.
   - `-o yaml`: This will output the resource definition in YAML format on screen.
   - Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes: `kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`. (This will automatically use the pod's labels as selectors, but you cannot specify the node port. You have to generate a definition file and then add the node port in manually before creating the service with the pod.)
 - Kubectl apply command(declarative):
   - local file
   - last applied configuration (json format): stored in an annotation of live object configuration named: `last-applied-configuration`
   - k8s: live object configuration
   - **note:** `kubectl create` or `kubectl replace` don't store `last-applied-configuration` like `kubectl apply`
##
