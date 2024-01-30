# certified-k8s-admin
## Table of Contents
 - [Certification Tip](#certification-tip)
 - [Core Concepts](#core-concepts)
 - [Scheduling](#scheduling)
 - [Logging and Monitoring](#logging-and-monitoring)

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

## Scheduling
 - manual scheduling: usually, the scheduler will decide which container goes to which node ( by adding `nodeName` property to pod configuration), but if you wanna schedule containers by yourself, then you can schedule the pod by adding `nodeName` property in the pod yaml config while creating the pod, or for the existing pod, create a pod `Binding` object and send a POST request to the pod binding API.
 - labels and selectors:
   - labels: can have multiple labels, and for `matchLabels`, make sure to add the labels so that the correct objects will be matched.
   - selectors: for commands, add option `--selector` with labels to get the objects expected.
   - annotations: used for attaching many different kinds of metadata to the objects, such as  build, release, client library or tool information, etc.
 - taints and tolerations:
   - taints: on the nodes
     - commands: `kubectl taint nodes <node name> key=value:taint-effect` : `key=value` is the taint, `taint-effect`: `NoSchedule | PreferNoSchedule | NoExecute ` (NoExecute will evict any existing pods)
   - tolerations: on the pods yaml configuration file `spec.tolerations` : `key:"app"`, `operator:"Equal"`, `value:"blue"`, `effect:"NoSchedule"`
   - **note: taints and tolerations are just restrictions, in other words, there's no guarantee that certain pods will go to certain nodes, unless we use node affinity**
   - `kubectl describe node kubemaster | grep Taint` : master node has a taint to prevent any pod from launching on.
 - node selectors: designate certain pods to go to certain nodes
   - for pods: add property `nodeSelector` with certain labels (key=value)
   - for nodes: use command `kubectl label nodes <node name> key=value`
   - **note:** can only handle simple use cases
 - node affinity: compared to `node selectors`, the `operator` has more features, such as `Exists`, `In`, `NotIn`, etc.
   - for nodes: use command `kubectl label nodes <node name> key=value` to add labels to nodes
   - for pods: add property section `affinity`
     - types: `requiredDuringSchedulingIgnoredDuringExecution` , `preferredDuringSchedulingIgnoredDuringExecution` and a planned one `requiredDuringSchedulingIgnoredDuringExecution`
     - explain: `duringScheduling` --> for any new pods, `IgnoreDuringExecution` --> for any existing pods
 - taints and tolerations vs node affinity: `taints and tolerations is used to make sure a certain node only takes certain pods, and node affinity is used to make sure that certain nodes have bindings with certain pods, meaning those pods would go into other nodes`
 - resource requirements and limits:
   - requests: resources required for running the pod
   - limits: the limits of amount of resources that a pod can use at maximal
   - ideally, set requests but not limits, but in some cases, limits should be set as well
   - `limitRange` object to setup requests and limits  --> for pods
   - `ResourceQuota` object to be created at namespace level --> to manage the total resource usage
 - A quick note on editing Pods and Deployments: for editing a pod specifications, some fields are not allowed to be updated during execution, however, those changes will be saved in a yaml file of `tmp` folder, so we can delete the current the pod, and then create a new pod using that saved file. The second option is using `kubectl get pod <pod name> -o yaml > <yaml name>.yaml` to extract yaml information, then update it, delete the current pod, and then using the new yaml file to create a new pod. While for deployment, directly using `kubectl edit deployment <deployment name>`, then you are good to go.
 - daemonsets: make sure there's one instance of pod existing on each node
   - use cases: monitoring solution & log viewer / kube-proxy / networking
   - a daemonset object configuration is like a replicaset
   - **note:** by far, `kubectl create` wouldn't be able to create a daemonset, but we can create a deployment first, then update the yaml file, and then create a daemonset.
 - static pods: the **kubelet** works on pod level, can only understand pod, other objects like replicaset, deployment are part of the whole k8s cluster architecture, which cannot be created without other components.
   - store yaml file at the path: `/etc/kubernetes/manifests`
   - config the path:
     - kubelet.service file: option: `--pod-manifest-path=`
     - kubelet.service file: option: `--config=<a yaml file>` with `staticPodPath: <path>`
   - `kubectl` knows the static pods(a mirror object), but cannot update/delete it, instead, it can only be deleted by modifying the file in the `path`
   - use cases: create a controlplane component as a static pod, which can be managed by kubelet, which is how kube admin tool sets up k8s cluster.
   - **note:** both `daemonset` and `static pod` are ignored by `kube-scheduler`
   - **remember the path `/var/lib/kubelet/config.yaml` --> staticPodPath where all static pod configuration files located**
 - multiple schedulers: we can create our own scheduler, and instruct kubernetes when to use which
   - create `kubeSchedulerConfiguration` object
   - deploy custom scheduler as a pod (when using `kubeadm`)
   - when there are multiple master node with a copy custom scheduler running within, then we may wanna set up property `LeaderElection( in scheduler yaml configure file)` to make sure there's only one scheduler gets running at a time.
   - to view shceduler, `kubectl get pods -n kube-system`
   - to use custom scheduler, make sure there's a `schedulerName` property defined in pod configuration file
   - to figure out which scheduler gets picked up:
     - view events: `kubectl get events -o wide`
     - view scheduler logs: `kubectl logs <scheduler name> -n kube-system`
 - configuring scheduler profiles:
   - when a pod is created, it will be put in a scheduling queue based on its priority (the `priorityClassName` property in the pod configuration), which also means that a `PriorityClass` object needs to be created first.
   - there are 4 stages: `scheduling queue`, `filter`, `sort`, `bind`. Each stage will use some plugins
   - for each stage, there are some extension points for extra or custom functionalities.
   - we can create multiple scheduler profiles in a single scheduler instead of creating multiple schedulers.

## Logging and Monitoring
