# k8s_learning_notes

This note is based on: https://www.youtube.com/watch?v=X48VuDVv0do

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
* The url for your service typically look like this: protocol://service-ip:application_port.
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



###  Deployment

* Deployment is a blueprint for your application(pod), you specify how many replicas you need.
* <u>In practice, you don't create pods, you create deployments</u>
* Abstraction of Pods
* However, Deployment cannot replicate database because it has a state, which is its data. Mechanisms needed to avoid data inconsistency(reason for **StatefulSet**)

### StatefulSet

* Handle syncronization
* StatefulSet in k8s is complex. Therefore, people usually deploy their database application outside k8s, and only deploy stateless application on k8s