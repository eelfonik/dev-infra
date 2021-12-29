## Kubernetes

https://www.youtube.com/watch?v=s_o8dwzRlu4

### Architecture

One **master node** with several **worker nodes** ( kubelet ), the (docker) containers live inside worker nodes

#### Master node 

the *master node* is running some controls process of the cluster:

- **API Server** ( which by itself is also a container ), which connect to some tools like UI (kubernetes dashboard), API, CLI. It's the entrypoint to k8s cluster.

- **Controller Manager**: keep track of what's happening inside

- **Scheduler**: schedual pods ( an abstraction of containers ), decides on which worker node to start new Pod, based on the available resources of each worker node, and the workload.

- **etcd** ( key-value storage ): k8s backing store, store the current state of cluster

> Since master node requires less resources, but is more important for a given cluster, in prod we ususally have multiple master nodes for the same cluster.

#### Worker Nodes

The *worker nodes* is where the application is running

#### virtual network

There's also a virtual network layer that connects all those nodes for them to work as a whole.



### Components

#### Pod

- Pod : smallest unit in k8s, create the running env for a container. It usually runs 1 app per pod. 
- Each pod has an (internal) IP created by the virtual network inside cluster, so they can communicate with each other ( EX: application pod communicate with the DB pod )
- Pods are ephemeral ( can die at anytime... ), if one pod is died, we'll create a new Pod with new IP address, which is very troublesome when it's happened

#### Service (Ingress)

- Internal services: 

To overcome the above mentioned problem ( Pod dies and IP address changed ), k8s can attach a **service** to each Pod, with *permanent IP address*. And the Pods will communicate with each other via these **services**. 

> So it means the service will track the died Pod & the newly created replacement Pod to attach to ?

- External services:

It's responsible to expose the IP address of a Pod to external (web) access, but we don't want to expose a http IP with numbers, so when we create external services, we'll go through **Ingress**, a component that will add the security layer (https) & format the IP to a readable one, means, instead of `http://xx.xxx.xxx.xx:8080`, we can have  sth like `https://my-app.com`, or in another word, *Ingress is responsible to route external traffic into cluster*. 

> N.B. the services are also acting like a **load balancer** to rout to different replicats Pods when used by **deployments**.

#### ConfigMap & Secret

The config of things like database url is usually stored inside the built application !

- ConfigMap

So if you want to change the database url, you'll need to re-build the image, push it to the repo, pull the new image to Pod, and restart.

To avoid this, k8s has a **ConfigMap** to store the *external configuration* of applications.

Every Pod will receive the configMap, and then it can use them.

> N.B. DON'T store sensitive info such as db password inside the configMap

- Secret

To be able to store the confidential external config, k8s uses `sercret` component. ( with base64 )

But be aware that the built-in security mechanism is not enabled by default, so you need to encrypt the infos yourself. (Check the official doc for more about secret component) 



#### Volume (Data storage)

To persist the data ( which is not k8s job )

So you can attach either a local storage ( in the same worker node that the Pod is running ), or a remote one ( outside of k8s cluster ) to a Pod.



#### Deployment

You can use deployment as a **blueprint** to specify how many replicats you want to have for a Pod.

In another word, you DON'T create Pods, you create **Deployments** ( act as a blueprint ).

> ❗️database Pod cannot be replicated by Deployment, because those Pods are connected to an external storage disk (be it local or remote ), or rather, those Pods have *states*. So we cannot simply replicate them, otherwise we'll have data inconsistent issue.

So we have another component **StatefulSet**, which is used to handle this kind of *stateful* Pods ( mainly database Pods ). BUT, handle this kind of Pods is not easy with k8s, so in practice, it's common to host DBs **outside of k8s clusters**. 



### Configuration

API server in the main/master node is the only entrypoint into the cluster. In order to configure how it works, we need to have a configuration file.

#### Configuration file structure

The conf is send using either YAML or JSON format, and it consists of multiple parts:

- **apiVersion**

- **kind** => choose one of the above components kind ( EX: `Deployment`, `Service`, etc)

- **metadata**

`name`: the name of the component

`labels`: 

- **spec**

all the specifications that we want, N.B. the `spec` is of course related with `kind` we specified above. 

- **status**

There's also *another part* of configuration called `status`, which is generated & added automatically by k8s. This `status` is used to check the **desired status** & the **actual status**. So if they don't match, then k8s will try to fix it.

> Q: So where did k8s get the status ?
>
> A: from the `etcd` key-value storage that we mentioned above :) 👆, this storage holds the current status of ANY k8s components.

#### Where to store ?

Normally we'll store it alongside with our codes



### Minikube & Kubectl

Let's have a check of a typical cluster setup:

- At least 2 master nodes
- Multiple worker nodes

All those nodes can sit at different virtual/physical machines

So it'll be painful to test this setup in your local machine. To be able to test this locally, we have a tool called **minikube**

#### minikube

In general, the idea is to run the master processes & the worker processes all in ONE node, and this node is shipped within a docker container ( 😂 俄罗斯套娃 ）

#### kubectl

This command line tool works together with minkube cluster, to talk to the **API server** in the master processe. 

But it can also work with the **real cloud cluster**.

