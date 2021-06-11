# k8s_learning_notes

This note is based on: https://www.youtube.com/watch?v=X48VuDVv0do

[TOC]

## Components

### Pod

* A pod is a smallest unit of K8s.

* It is an abstraction over container. K8s do this because they want you to only interact with the kubernetes layer and don't need you to know about containers
* usually 1 application per pod
* each pod gets its own IP address
* new ip address on re-creation(pods are ephemeral)
  * this is not ideal and inconvenient(reason for **Service**)

### Service

* Service is a static/permanent IP address that can be attached to each pod
* Service is also a load balancer
* There are external and internal services
  * External services communicate with public world, i.e. hosting website
  * Internal services, i.e. database
* The url for your service typically look like this: protocol://service_ip:application_port.
  * You don't want users to access your web like this :).(reason for **Ingress**)

### Ingress

* Ingress bascially forward the request to the service

Now, consider this scenario:

You store your database url in your application code, i.e. mongo_database. Later, the url is changed. This means you'll need to modify your source code in the docker image and redeploy the pod. NOT GOOD.(reason for **ConfigMap**)

### ConfigMap

* External configuration of your application
  * For example, put database_url in the config
* Connect the ConfigMap to the pod, then pod can get those config data
* Feels like a seperate tiny database for your application
* You don't store secret data in Config Map.(reason for **Secret**)

### Secret

* Secret is basically an encoded ConfigMap

### Volumes

* Both local and remote storage can be attached to a pod



<u>**Of course you want replicas**</u>



### Deployment

* Deployment is a blueprint for your application(pod), you specify how many replicas you need.
* <u>In practice, you don't create pods, you create deployments</u>
* Abstraction of Pods
* However, Deployment cannot replicate database because it has a state, which is its data. Mechanisms needed to avoid data inconsistency(reason for **StatefulSet**)

### StatefulSet

* Handle syncronization
* StatefulSet in k8s is complex. Therefore, people usually deploy their database application outside k8s, and only deploy stateless application on k8s

## Architecture

### Node processes

* Each Node has multiple Pods on it
* There are **3** processes that must be installed on every Node. These 3 processes schedule and mange the pods
  1. container runtime, i.e. docker
  2. kubelet
     * Kubelet interacts with both the container and the node itself
     * Kubelet starts the pod with a container inside. Assigning resrouces from the nodes to pods
  3. Kube Proxy
     * KP forwards the requests to the approriate pod through Service

### Mater nodes

* There are **4** processes that runs on every master node

  1. Api server

     * Cluster gateway(A single entry point)
     * Gatekeeper for authentication

     * This is what your kubectl cmds is interacting with

  2. Scheduler

     * Schedule pods on worker nodes
     * **Scheduler just schedule where the pod should be. Kubelet is the one that really starts the pod**

  3. Controller manager

     * Detects cluster state changes, i.e. pods die
     * After detection, ask scheduler to replace the pod for example

  4. etcd

     * A Key Value store of all the cluster changes
     * **It's the cluster brain**
     * Of course, the application data is not stored in etcd

## Basic kubectl commands

- kubectl create deployment <name> <option>
- kubectl edit deployment <deployment-name>
  - This will open up a config file of the certain deployment to enable editing
- kubectl delete deployment <deployment-name>
  - This will remove all the pods created by this deployment, including all replicas
- kubectl apply -f <config-file>
  - It can create or update the component

### Help Debugging

* kubectl logs <pod-name>
* Kubectl get pod -o wide
  * Provides little bit more info about the pod

- kubectl describe pod <pod-name>
  - Shows the detailed status of a pod

* Kubectl exec -it <pod-name> -- bin/bash
  * Get interactive terminal of the pod 

## YAML Configuration File

Each config file has 3 parts

1. metadata
2. specification
   * Attributes of spec are specific to the **kind**, i.e. Deployment or Service
3. status
   * Auto generated by k8s
   * This comes from etcd of the master node :)

### Connecting Services to Deployments

We use **label** under **metadata** to label the resource with any key-value pair. Then we use **selector** under **spec** to connect to a **label**

![image-20210611110906689](/Users/weisy/Library/Application Support/typora-user-images/image-20210611110906689.png)

We need to match ports too

![image-20210611111207128](/Users/weisy/Library/Application Support/typora-user-images/image-20210611111207128.png)

