# CKAD Preparation

Exam: https://www.cncf.io/certification/ckad/
Course: https://www.udemy.com/course/certified-kubernetes-application-developer/

## **Master Node**

Another node with Kubernetes in it, responsible for the orchestration of worker
nodes within a cluster. It embeds the Kube API Server.

### API Server

Frontend for Kubernetes. User management devices, CLI, etc. talk directly with
API server to interact with a Cluster.

### `etcd` Key Store

Distributed reliable key-value store used by Kubernetes to store all data used
to manage the cluster. When you have multiple nodes and master, etcd stores all
that information in all the nodes of the cluster in a distributed manner.

It also implements logs within a cluster to ensure no conflict between masters.

### Controller

Responsible for noticing and responding when nodes, containers, endpoints goes
down, etc. - it makes decision to bring up new containers, in such cases.

### Scheduler

Responsible for distributing work, or containers, across multiple nodes. Assign
newly-created containers to nodes.

## **Worker Nodes**

Where the containers are hosted, thus make use of a Container Runtime.
To communicate with the Kube API Server from the Master node, the worker nodes
embeds the kubelet service

### kubelet

Agent that runs on each node in the cluster, responsible for making sure the
containers are running in the nodes as expected. Communicates with the Kube API
server from the Master node.

### Container Runtime

Underlying software used to run containers. Docker runtime is the most common
one, but there are a few others:
* [rkt](https://www.redhat.com/pt-br/topics/containers/what-is-rkt): maintained by RedHat
* [CRI-O](https://cri-o.io/): Cloud Native Computing Foundation incubating project

![Master and Worker nodes](./images/master-worker.png)

## Kubectl CLI

Deploy and manage applications in a Kubernetes cluster:
* Cluster information: `kubectl cluster-info`
* Status of other nodes: `kubectl get nodes`
* Manage other resources

Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

------

# Components

## Pod

Pod's are the smallest working units of Kubernetes which runs a single-instance
of an application.

![Simple pod overview](./images/pod.png)

Pods run on top of a Node. When a Node does not have available resources to
deploy new pods, then the scheduler deploys a new Node to run that pod.

Pods usually have a 1:1 relationship with containers. To scale up an application,
one must create new pods, rather than new containers to an existing pod.

Load balancer between the containers is handled by a separate entity.

## Multi-Container Pods

Even though pods and container usually have a 1:1 relationship, it is not
strictly enforced, thus we can have multiple containers running in a single pod.

One scenario for this, would be helper containers that might be doing some kind
of supportive task for a web application, i.e: processing a file uploaded by the
user, etc. In that case, both containers are part of the same pod and share the
same network space (`localhost`) - they can easily share the same storage space
as well.

![Multi-container pods](./images/multi-container-pods.png)

One common application, would be [Envoy](https://www.envoyproxy.io/) sidecar,
which is a reverse-proxy that [istio](https://istio.io/) injects in
application's pods.

![Envoy sidecar in istio](./images/istio-envoy.png)

### Deploying via Kubectl imperatively

There are two ways of deploying a pod using `kubectl` CLI, the first one is
imperatively using `kubectl run` passing as argument the image of the container.
What it does is first create the pod, then fetches the image from
docker hub registry, which get's finally deployed as a container inside the pod.

```bash
$ kubectl run nginx --image=nginx
pod/nginx created
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          6s
```



### Deploying via Kubectl declaratively

Declarative deploys need a YAML file specifying the resource that is going to
be created. To create via `kubectl` we need to `apply`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    type: webserver
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
```

```bash
$ kubectl apply -f ./examples/nginx-pod.yaml
pod/nginx created
$ kubectl get pods
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          15s
```

### Deleting

To delete a pod via `kubectl`:

```bash
$ kubectl delete pod nginx
pod "nginx" deleted
```
