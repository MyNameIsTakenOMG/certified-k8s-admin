# certified-k8s-admin
## Table of Contents
 - [Certification Tip](#certification-tip)
 - [Core Concepts](#core-concepts)
 - [Scheduling](#scheduling)
 - [Logging and Monitoring](#logging-and-monitoring)
 - [Application LifeCycle Management](#application-lifecycle-management)
 - [Cluster Maintenance](#cluster-maintenance)

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
 - monitor cluster components
   - monitor: `Metrics Server`: one instance per cluster, and in-memory solution. in `kubelet`, there's sub component `container advisor` used to gather metrics from pods and then make them available for `Metrics Server`. To install `Metrics Server`(minikube or other...)... To view metrics, `kubectl top node`, `kubectl top pod`
 - managing application logs: `kubectl logs -f <pod name> <explicitly specify container name if there are more than one>`

## Application LifeCycle Management
 - rolling updates and rollbacks
   - rollout and versioning: when creating a new `deployment`, a new `rollout` gets triggered which will create a new `revision`, which helps us keep track of the changes made to our deployment and enables us to roll back to the previous version if needed.
   - command: `kubectl rollout status deployment/<deployment name>` / `kubectl rollout history deployment/<deployment name>`
   - deployment strategy: recreate(application downtime), rolling updates(default)
   - kubectl apply: `kubectl apply -f <deployment filename>` or `kubectl set image deployment/<deployment name> <image name and version>`
   - rollback: `kubectl rollout undo deployment/<deployment name>`
 - application commands: `docker file` `CMD` and docker command: `ENTRYPOINT`, `CMD`
 - application-commands and arguements:  in kubernetes env, or pod definition,  `ENTRYPOINT` --> `spec.containers.command`, `CMD` --> `spec.containers.args`
 - configure environment variables in applications: under the `spec.containers.env`, we can have name-value pairs for env variables, or using `configmap`, `secret` objects refs
   - configmap: can be injected entirely into pod using `envFrom`
     - create imperatively: `kubectl create configmap <configmap name> --from-literal=<key>=<value> --from-literal=...` or `kubectl create configmap <name> --from-file=<path>`
     - create declaratively: `kubectl create -f`
     - configmaps in pods:
       - entire configmap: `envFrom`
       - single env: `env`
       - volumes: `volumes.configmap`
 - configure secrets in applications:
   - create secrets imperatively: similar to configmap
   - create secrets declaratively: similar to configmap
   - encode secrets: `echo -n 'password' | base64 `
   - view secret: `kubectl describe secret <secret name>` , `kubectl get secret <secret name> -o yaml`
   - decode secret: `echo -n 'password' | base64 --decode`
   - secrets in pods: similar to configmap, entire file, single value, volume
   - **note:** secrets are not encrypted. Don't checkin secret in source control management (like github). considering encrypt secrets in ETCD. Anyone in the same namespace can access the secrets, considering using RBAC. Considering using 3rd parth secret provider, like AWS, Azure or GCP.
 - demo: encrypt secrets at rest --> the point is to check if `kube-apiserver` already enabled the `--encryption-provider-config`, otherwise create a `encryptionconfiguration` file, mount this file as a volume onto `kube-apiserver` pod config file.
 - multiple containers pods:
 - Multi-container Pods Design Patterns: There are 3 common patterns, when it comes to designing multi-container PODs. The first and what we just saw with the logging service example is known as a side car pattern. The others are the adapter and the ambassador pattern. But these fall under the CKAD curriculum and are not required for the CKA exam. So we will be discuss these in more detail in the CKAD course.
 - InitContainers: at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where `initContainers` comes in. An `initContainer` is configured in a pod like all other containers, except that it is specified inside a `initContainers` section. When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts. You can configure multiple such initContainers as well, like how we did for multi-containers pod. In that case each init container is run `one at a time in sequential order`. If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
 - Self Healing Applications: Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times. Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.

## Cluster Maintenance
 - OS upgrades: if a node is down within 5 min, then `kubelet` process starts, and pods come back online. But if a node is down for more than 5 min, then pods will be gone. If the pods in the node are part of a replicaset, then some new pods will be created in other nodes. The `--pod-eviction-timeout` flag is deprecated from 1.27 kubernetes release. From now on, flags `--default-not-ready-toleration-seconds` and `--default-unreachable-toleration-seconds` are used through kube-apiserver pod to control the pod eviction. So a safer way to do maintenance is the use `kubectl drain` to "move" pods to other nodes, remember that the node is unscheduled, you have to manually make it schedulable by using `kubectl uncordon`. Accordingly, there's command `kubectl cordon` that simply mark a node unscheduled, unlike `kubectl drain`, the pods will not be "moved".
 - kubernetes releases: apart from `ETCD cluster` and `CoreDNS`, other components are sharing the same version number.
 - **note:** if `kube-apiserver` version is "X", then `controller-manager` and `kube-scheduler` version should be X-1 or X, and `kubelet` and `kube-proxy` version should between X-2 and X, and `kubectl` version should between X-1 and X+1
 - cluster upgrade process:  kubernetes always maintain 3 minor versions. and the recommended way to upgrade is to upgrade 1 minor version at a time. using `kubeadm` to perform upgrade.
   - version number expain:  4.2.1 --> 4 major , 2 minor , 1 patch
   - two steps to upgrade the cluster: 1. upgrade the master node. 2. upgrade the worker nodes
   - strategies for worker nodes: 1.upgrade all worker nodes all together( facing downtime) 2. upgrade one at a time. 3. add new version nodes (using cloud platform to easily perform the operations).
   - Kubeadm - upgrade: `kubeadm upgrade plane`,  `kubeadm upgrade apply`  **kubeadm doesn't install or upgrade kubelet**
     - `apt-get upgrade -y kubeadm=1.12.0-00`, then `kubeadm upgrade apply v1.12.0` --> master node components version upgraded
     - `apt-get upgrade -y kubelet=1.12.0-00` to upgrade the version of `kubelet` for the master node
     - `systemctl restart kubelet` for the master node
     - **now for worker nodes:** `kubectl drain <worker node name>`, then `apt-get upgrade -y kubeadm=1.12.0-00`, then `apt-get upgrade -y kubelet=1.12.0-00`, `kubeadm upgrade node config --kubelet-version v1.12.0`, then `systemctl restart kubelet`, then `kubectl uncordon <worker node name>`
   - demo -- upgrade cluster (more up-to-date): 
 - backup and restore:
   - backup candidates:
     - resource configuration: `kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml` or tools like Velero can do the backup for you
     - ETCD cluster: etcd.service -->  `--data-dir=` the path where all data are stored. `ETCDCTL_API=3 etcdctl snapshot save <path+ 'snapshot.db'>`. `ETCDCTL_API=3 etcdctl snapshot status <path+ 'snapshot.db'>`. For restoring, first `service kube-apiserver stop`, then `ETCDCTL_API=3 etcdctl snapshot restore <path+ 'snapshot.db'> --data-dir=<path for new etcd cluster data>`, then configure etcd.service to use the new path to store data, then `systemctl daemon-reload`, `systemctl restart etcd`, `systemctl restart kube-apiserver`. 
     - Persistent volumes
   - `etcdctl` is a command line client for `etcd`. `export ETCDCTL_API=3`. if you want to take a snapshot of etcd, use: `etcdctl snapshot save -h` and keep a note of the mandatory global options. Since our ETCD database is TLS-Enabled, the following options are mandatory: `--cacert` verify certificates of TLS-enabled secure servers using this CA bundle. `--cert` identify secure client using this TLS certificate file. `--endpoints=[127.0.0.1:2379]` This is the default as ETCD is running on master node and exposed on localhost 2379. `--key` identify secure client using this TLS key file. Similarly use the help option for snapshot restore to see all available options for restoring the backup. `etcdctl snapshot restore -h`
   - This means that ETCD is set up as a `Stacked ETCD Topology` where the distributed data storage cluster provided by etcd is stacked on top of the cluster formed by the nodes managed by kubeadm that run control plane components.
   - go describe kube-apiserver to find details about controlplane components, `kubectl describe pod <kube api server pod name> -n kube-system` : especially `etcd`
   - remember `ps -ef | grep etcd` to get the running process
   - remember ` scp <path1> <path2>` --> copy files from localhost to a remote server
   - remember `/etc/systemd/system/etcd.service`
   - remember `/var/lib/kubelet/config.yaml` --> `staticPodPath`
   - **remember to redo the 'backup & restore #2' later**
##











