# Kubernetes Notes

Sources:
- [Kubernetes Course - Full Beginners Tutorial](https://youtu.be/d6WC5n9G_sM)
- [Kubernetes Crash Course for Absolute Beginners](https://youtu.be/s_o8dwzRlu4)
- [Kubernetes Tutorial for Beginners](https://youtu.be/X48VuDVv0do)

Kubernetes (K8s) is an open-source container orchestration system. It lets you manage multiple containers in different deployment environments, whether physical machines, virtual machines or cloud environments.

The rise of microservices caused the increased use of container technologies.

Orchestration tools, like Kubernetes, offer:
- high availability or no downtime
- scalability or high performance
- disaster recovery, backup and restoration

Kubernetes takes care of:
- automatic deployment of containerized applications across different servers
- distribution of the load across multiple servers
- auto-scaling of the deployed applications
- monitoring and health check of the containers
- replacement of failed containers

Kubernetes supports multiple container runtimes, including:
- Docker
- CRI-O
- containerd

## Kubernetes Architecture
A Kubernetes cluster is made up of one **master node** with a number of **worker nodes** connected to it. Each node has a kubelet process running on it, which makes it possible for the nodes in the cluster to talk to each other.

Each worker node has containers of different applications deployed on it. Your applications run on these worker nodes.

The master node runs several Kubernetes processes that are necessary to manage the cluster. One of these processes is an **API Server**, which is also a container. This is the entrypoint to the Kubernetes cluster and the process that the different Kubernetes clients (UI, API, CLI) will talk to. 

Another process running on the master node is the **Controller Manager**, which keeps track of what's happening in the cluster. There's also the **Scheduler**, which decides which node a new pod should be scheduled on, based on the available resources.

Another important component of the cluster is an **etcd** key-value store, which holds at any time the current state of the cluster. Backup and restore is made from this etcd snapshot.

The **virtual network** allows the master ndoe and worker nodes to communicate with each other. It spans all the nodes that are part of the cluster, turning them into one unified machine.

Individual worker nodes will be bigger and require more resources than the master node, since they have a higher workload. In production environments, you will usually have at least two master nodes in case of failure.

## Main Kubernetes Components
### Pods
A **pod** is the smallest unit in Kubernetes. A pod is an abstraction over a container. Kubernetes wants to abstract away the container runtimes so you can replace them if desired. You only interact with the Kubernetes layer.

In Docker, a container is the smallest unit. Containers are created inside pods. Usually there is only one application per pod.

Kubernetes offers a **virtual network** out of the box. Each pod gets its own (internal) IP address and they can communicate with each other using this IP address.

Pods are ephemeral and can easily die. When this happens a new one (with a new IP address) will get created in its place.

A pod contains one or more containers, shared volumes and shared network resources (such as a shared IP address). All containers in the same pod share namespaces, volumes and IP addresses.

A single container per pod is the most common use case. But if containers tightly depend on each other and could exist in the same namespace it's possible to have several containers in the same pod.

Each container in a pod must exist on the same (virtual) server. 

### Service and Ingress
A **service** is a permanent IP address that can be attached to each pod. The lifecycles of services and pods are not connected, so if a pod dies its IP address will remain due to the service.

For your application to be available through the broswer, you have to create an external service.

For things that you don't want to be open to public requests, such as a database, you would create an internal service.

You specify the type of service on creation, with internal being the default type. 

**Ingress** routes traffic into the cluster. It lets you add a secure protocol and custom URL before the request is forwarded to the service.

### ConfigMap and Secret 
 Usually config values, such as a database URL, are in the built application, so you would have to rebuild and redeploy the application image if this config changed.  

The **ConfigMap** component provides the external configuration for your application, containing data such as database URLs. In Kubernetes, a pod can directly access ConfigMap to get this configuration data. 

ConfigMap is for non-confidential data only. Don't put credentials there.

The **Secret** component stores confidential data in base64 encoding. This component is meant to be encrypted using third-party tools. Pods can then read this data from Secret.

You can access data from ConfigMap or Secret inside your application using environmental variables or a properties file. 

### Volumes
**Volumes** let you persist data longterm even as pods come and go. Volumes attach physical storage on a hard drive to your pod. This storage could be on a local machine or remote. In either case, it is outside the Kubernetes cluster. The cluster itself explicitly doesn't manage data persistence.

### Deployment and StatefulSet
Instead of relying on one application pod, Kubernetes replicates everything on multiple servers to avoid downtime.

The replica is connected to the same service. The service is also a load balancer and will forward requests to the least busy pod.

A **deployment** defines a blueprint for pods, specifying how many replicas you want. In practice, you create deployments, rather than creating pods directly. Deployments are an abstraction on top of pods.

Databases can't be replicated using deployments because they have state. A **StatefulSet** takes care of replicating database pods while making sure that database reads and writes are synchronized so that no inconsistencies are offered. 

Deploying StatefulSets is not easy and so databases are often hosted outside the Kubernetes cluster.

### Clusters and Nodes
A Kubernetes cluster consists of nodes. A node is a (usually virtual) server. Nodes can be located in different data centers but are usually located close to each other. Inside nodes are pods and inside pods are containers.

In a cluster, there is a master node and worker nodes. The master node manages other nodes, for example distributing load across them. All pods related to your application are deployed on worker nodes. 

The master node runs only system pods, which are responsible for the work of the cluster in general. The master node is also the **control plane**.

Services, such as kubelet, kube-proxy and container runtime, are present on each node. 

Every node has the following processes:
- the container runtime
- kubelet, which interfaces between the container and the node, starting a pod with a container inside
- kube-proxy, which forwards requests between pods and containers

Master nodes run the following processes:
- API Server, the cluster entrypoint and gatekeeper
- Scheduler, assigns pods to the least busy worker nodes
- Controller Manager, detects cluster state changes and requests pods to be rescheduled
- etcd, a key-value store of the cluster's state, recording every change in the cluster

On the worker nodes:
- kubelets communicate with the API Server service on the master node.
- kube-proxy is responsible for network communication inside each node and between nodes.

On the master node:
- The API Server service is the main point of communication between different nodes.
- The Scheduler service on the master node is responsible for planning and distributing the load between nodes in the cluster.
- Kube Controller Manager is the single point which controls everything in the cluster and what happens on each node in the cluster.
- Cloud Controller Manager interacts with the cloud service provider where you run your cluster.
- etcd stores all logs related to the operation of the cluster as key-value pairs.
- A DNS service is responsible for name resolution in the cluster, letting you connect different deployments with each other.

There are usually multiple master nodes for redundency. The API Server is load balanced and etcd forms a distributed storage across all master nodes.

### Layers of Abstraction
- **Deployment** manages a ReplicaSet
- **ReplicaSet** manages a Pod
- **Pod** is an abstraction of a Container
- **Container** is the lowest level

Everything below the deployment should be managed automatically by Kubernetes.

## Kubernetes Configuration
All configuration in a Kubernetes cluster goes through the master node via the API Server. UIs, APIs and CLIs send their configuration requests to the API Server, which is the only entrypoint into the cluster. These requests have to be in either yaml or json format.

Configuration in Kubernetes is declarative. You specify what state should exist and Kubernetes tries to match that.

Every configuration file in Kubernetes has three parts:
- metadata
- specification
- status

The attributes in the spec are specifc to the kind of component you're creating.

The status is automatically generated and added by Kubernetes. If the desired and actual statuses don't match, Kubernetes will work to self-heal. 

Kubernetes updates state continuously. This information comes from etcd, which holds the current status of any Kubernetes component at all times.

Configuration files are usually version controlled and stored with your code.

The template is essentially a configuratin file inside a configuration  file and has its own "metadata" and "spec" sections. This configuration applies to a pod, providing it with a name, image and port.

Deployments and Services are connected through labels and selectors. Metadata contains the labels, while spec contains the selectors.

Deployments are connected to pods via labels. This label is matched by the selector.

When you define a service, its selector points to the deployment's label, and so also to its pods.

## kubectl
kubectl is a command line tool that lets you connect to and manage clusters remotely. 

`kubectl --help` will list all available commands.

## minikube
In a production setting, you would have at least two master nodes and multiple worker nodes.

minikube lets you locally create a Kubernetes cluster with a single node for learning and development purposes. In minikube, master processes and worker processes both run on one machine with a Docker runtime pre-installed.

Use `minikube start --driver=VIRTUAL_MACHINE` to start your cluster using a specific virtual machine or container manager, such as VirtualBox or Hyperkit.

Use `minikube ip` to check which IP address was assigned to the virtual machine running your Kubernetes cluster.

`minikube ssh` lets you ssh into the minikube cluster. Then run `docker ps` to see the running containers. You will see containers for the API server, scheduler, kube-proxy and more.

kubectl is not available inside the node. Outside the node, use `kubectl cluster-info` to see if the control plane and coreDNS are running.

`kubectl get nodes` will display the nodes in the cluster. With minikube, a single node will act as both control-plane and master.

`kubectl get pods` will display pods in the default namespace.

`kubectl get namespaces` will display namespaces, which are used to group different resources and configuration objects.

`kubectl get pods --namespace=NAMESPACE` will display pods in a specific namespace.

`kubectl run POD_NAME --image=IMAGE` will create a single pod based off a specfied image (by default fom Docker Hub). 

`kubectl describe pod POD_NAME` will describe a specific pod.

`kubectl logs POD_NAME` will get the logs for a specific pod.

In order to be able to connect to pods, you have to create services in Kubernetes. You can't just use their internal IP address.

If you look for containers, you'll see that a container also has a pause container used to lock the namespace of a specific pod. If there are several containers in the pod, they share the same IP address.

While you are sshed into Minikube, you can use `docker exec -it CONTAINER_ID sh` to connect to a container.

`kubectl delete pod POD_NAME` will delete a pod, including all volumes and namespaces.

You can alias `kubectl` to `k` with `alias k="kubectl"`.

## Deployments
The most common way to create and scale pods is to use deployments. All pods inside a deployment will be the same. You can create multiple copies of the same pod and distribute the load across different nodes in the Kubernetes cluster.

`kubectl create deployment DEPLOYMENT_NAME --image=IMAGE` will create a new deployment with a single pod.

`kubectl get deployments` (or `kubectl get deploy`) will list deployments.

`kubectl describe deployment DEPLOYMENT_NAME` will provide details about the deployment, such as labels, annotations and selectors. 

`kubectl delete deployment DEPLOYMENT_NAME` will delete a deployment.

`Selector`s are used to connect pods with deployments, since they are seperate objects in Kubernetes. There will be a corresponding `Label` in the pod.

`Replicas` will tell you the quantity of pods desired by the deployment, as well as the quantity that are currently running. `StrategyType` specifies how to perform updates of deployments. 

`ReplicaSet` is a set of replicas of your application, which manages all pods related to the deployment. When you create a deployment, you will see that a replica set was scaled up to 1, meaning that one pod was created. ReplicaSet acts as a layer between deployment and pod.

The name of a pod starts with the name of the replica set and then a specific hash for the partcular pod, such as `nginx-deployment-84cd76b964-tw79f`. Multiple pods that belong to the same replica set will have the same prefix.

`kubectl describe pod POD_NAME` will prvide details about the particular pod.

`kubectl scale deployment DEPLOYMENT_NAME --replicas=QUANTITY` will scale the number of pods to the specfied quantity. All these pods will belong to the same replica set. Each pod will have a dfferent IP address. If you were running multiple nodes, the load would be distributed across different nodes.

Running `kubectl scale deployment DEPLOYMENT_NAME --replicas=QUANTITY`  again with a different quantity lets you scale the deployment up or down.

### Services
In Kubernetes you need to create services if you want to connect to specfic deployments using specific IP addresses.

You could create a ClusterIP. This IP address will be created and assigned to a specific deployment. You will be able to connect to this deployment only from **inside** the cluster using this virtual IP address. 

You could also create external IP addresses, to expose the deployment to the outside world. It's possible to expose the deployment to the IP address of the node, or to use a load balancer.

The most common solution is to have a load balancer IP address, which serves an entire cluster for a specific deployment. You will be able to connect to your deployment no matter where the pods are created using a single IP address. These load balancer IP addresses are usually assigned by a cloud provider, like AWS or GCP.

Use `kubectl expose deployment DEPLOYMENT_NAME --port=EXTERNAL_PORT --target-port=INTERNAL_PORT` to map an internal port to an external port.

`kubectl get services` (or `kubectl get svc`) will list services.

`kubectl describe service SERVICE_NAME` will provde details about a specific service. The diffferent endpoints listed describe different pods which are used for connections to this particular ClusterIP.

Use `kubectl delete deployment DEPLOYMENT_NAME` to delete a deployment and `kubectl delete service SERVICE_NAME` to delete a service.

If you create a deployment with a NodePort service `kubectl expose deployment DEPLOYMENT_NAME --type=NodePort --port=PORT`, you will be able to access it via the node's IP address and the automatically generated port.

`minikube service SERVICE_NAME` will open the service in your default browser.

If you create a deployment with a LoadBalancer service
`kubectl expose deployment DEPLOYMENT_NAME --type=LoadBalancer --port=PORT`, a load balancer will be assigned automatically by the cloud service provider.

## Docker Hub
Once an image has been deployed to Docker Hub, it can be used to create a Kubernetes deployment.

`kubectl create deployment DEPLOYMENT_NAME --image=IMAGE_NAME`

### Updating Deployments
The RollingUpdate strategy means that new pods will be created with the new image while exisiting pods are still running. Pods will be replaced one by one until all pods are running the new image.

Use `kubectl set image deployment DEPLOYMENT_NAME DEPLOYMENT_NAME=NEW_IMAGE_NAME` to update the image being used by the deployment.

`kubectl rollout status deploy DEPLOYMENT_NAME` lets you check the status of the update.

Use `kubectl set image deployment DEPLOYMENT_NAME DEPLOYMENT_NAME=OLD_IMAGE_NAME` to roll back to a previous image.

If you delete an individual pod, Kubernetes will create a new pod to keep the number of pods the same as the replication number.

### Kubernetes Dashboard
In minikube, you can launch the Kubernetes dashboard with `minikube dashboard`. In a production environment, you would need to manage secure access to the dashboard. 

The dashboard provides information about workloads, services, configuration, clusters, and more.

## YAML Configuration
It's best to use a declarative approach where you create yaml configuration files which describe the details of the deployments and services you desire.

The Kubernetes VS Code extension lets you quickly scaffold yaml configuration files.

The `matchLabels` key specifies which pods will be managed by a deployment. The `template` key describes the actual pod. The `spec` key specifies which containers will be created in the pod. `resources.limits` specifies memory and CPU limits for each pod created by this deployment.

When you have multiple replicas of a pod, each pod gets a unique name but they share the same label to group them together.

The [Kubernetes docs](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/) contain more details on configuring deployments.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
	name: k8s-web-hello
spec:
	selector:
		matchLabels:
			app: k8s-web-hello
	template:
		metadata:
			labels:
				app: k8s-web-hello
		spec:
			containers:
			- name: k8s-web-hello
				image: gerhynes/k8s-web-hello
				resources:
					limits:
						memory: "128Mi"
						cpu: "500m"
				ports:
				- containerPort: 3000
```

You can create a deployment from this file using `kubectl apply -f deployment.yaml`

You can do the same for a service with `kubectl apply -f service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
	name: k8s-web-hello
spec:
	type: LoadBalancer
	selector:
		app: k8s-web-hello
	ports:
	- port: 3000
		targetPort: 3000
```

You can also delete deployments and services using `kubectl delete -f deployment.yaml -f service.yaml`.

## Connecting Deployments
You can connect to another deployment by referencing the name of a service.

## Modifying Container Runtime
You can specify a container runtime, such as Docker or CRI-O, when creating a minikube cluster.

`minikube start --driver=VIRTUAL_MACHINE --container-runtime=RUNTIME`

## Kubernetes Namespaces
A namespace is like a virtual cluster inside a cluster, allowing you to organize resources.

When you create a cluster, Kubernetes automatically creates a number of namespaces:
- default
- kube-node-lease
- kube-public
- kube-system
- kubernetes-dashboard (specific to minikube)

**kube-system** contains system processes. Do not create or modify anything here.

**kube-public** contains publicly-accessible data, such as a configmap containing cluster information, which is accessible without authentication.

**kube-node-lease** holds information about the heartbeats of nodes. Each node has an associated lease object in the namespace, which determines the availability of the node.

**default** contains the resources you create, if you don't create a new namespace.

`kubectl create namespace MY_NAMESPACE` creates a new namespace.

You can also create namespaces using a namespace configuration file.

It's a good practice to group resources by namespace, for example having a database namespace for the database and all its required resources.

The Kubernetes docs don't recommend namespaces for smaller projects, but they do help with organization.

If you have multiple teams working on the same applicaton, you'll need to use namespaces. Otherwise they might create deployments with the same name and overwrite each other.

Namespaces can also be useful for having staging and development environments in the same cluster.

Namespaces are also useful for blue/green deployments, where you have two different versions of production, the currently active version and the next version. 

Namespaces can be used to limit access and resources when working with multiple teams. You can give a team access to only their namespace. You can also limit the resources a namespace consumes, in terms of CPU, RAM and storage.

Namespaces help you:
1. Structure your components
2. Avoid conflicts between teams
3. Share services between different environments
4. Enforce access and resource limits per namespace

When using namespaces, consider that:
- You can't access most resources from another namespace. For example, each namespace must define its own ConfigMap and Secret.
- You can access services in another namespace, but you need to access them using `SERVICE_NAME.NAMESPACE`.
- Some components cannot be created within a namespace, such as persistent volumes. You can list these components using `kubectl api-resources --namespaced=false`

By default, components are created in the "default" namespace. 

To create components in a namespace, you can use `kubectl apply -f FILENAME.yaml --namespace=NAMESPACE`.

You can also specify the namespace in the configuration file.

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
	name: mysql-configmap
	namespace: my-namespace
data:
	db_url: mysql-service.database
```

To get a component in a specific namespace, you need to add the `-n` flag.

`kubectl get configmap -n NAMESPACE`

Creating namespaces in configuraton files is better as it leaves the process better documented and is more convenient for automated deployment.

You can change the active namespace with kubens, a seperate tool from kubectl.

`kubens` will give you a list of all namespaces and highlight which one is active.

`kubens NAMESPACE` will change the active namespace.

## Kubernetes Ingress
One way to access applications from the broowser is via an external service. This exposes an IP address and port.

Ingress handles external requests so that internal IP  addresses and ports are not opened to the world. Ingress will redirect requests to an internal service.

### External Service Configuration
```YAML
apiVersion: v1
kind: Service
metadata:
	name: myapp-external-service
spec:
	selector:
		app: myapp
	type: LoadBalancer
	ports:
		-  protocol: TCP
			port: 8080
			targetPort: 8080
			nodePort: 35010
```

### Ingress Configuration
```YAML
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
	name: myapp-ingress
spec:
	rules:
	- host: myapp.com
		http:
			paths:
			- backend:
				serviceName: myapp-internal-service
				servicePort: 8080
```

Ingress' routing rules specify that requests must be forwarded from a specified host to an internal service. `paths` specifies the URL path after the domain name. 

`host`should be mapped to the Node's IP address, which is the entrypoint.

### Internal Service Configuration
```YAML
apiVersion: v1
kind: Service
metadata:
	name: myapp-internal-service
spec:
	selector:
		app: myapp
	ports:
		- protocl: TCP
			port: 8080
			targetPort: 8080
```

Configuring Ingress isn't enough. You need an Ingress implementation, called Ingress Controller. This is a pod , or pods, that runs on the cluster and does evaluation and processing of Ingress rules.

Ingress Controller evaluates the rules you have defined in your cluster and manages redirections. It serves as the entrypoint in the cluster for all requests to the domain/subdomain that you've configured, and determines which forwarding rule should be applied.

There are multiple third-party Ingress Controllers to choose from. Kubernetes themselves offer K8s Nginx Ingress Controller.

You need to consider the environment on which the Kubernetes cluster is running. 

If you're using a cloud service provider with their own out of the box K8s solutions or virtualized load balancer, requests will hit this cloud load balancer first, which will redirect requests to the Ingress Controller pod. (You can configure this in different ways). The advantage of this is that you don't need to implement a load balancer yourself.

If you're deploying your cluster on a bare metal environment, you would need to configure an entrypoint, either inside the cluster or outside as a seperate server. This could be a proxy server with a public IP address and open ports. None of the servers in your cluster would be accessible from the outside. The proxy server would redirect requests to the Ingress Controller, which will decide which Ingress rule applies to that request.

In minikube, you can install an Ingress Controller with `minikube addons enable ingress`. This automatically starts the K8s Nginx Ingress Controller implementation.

If you run `kubectl get pod -n kube-system`, you will see the nginx-ingress-controller pod running in your cluster.

For example, to create an ingress rule for Kubernetes dashboard:

```YAML
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
	name: dashboard-ingress
	namespace: kubernetes-dashboard
spec:
	rules:
	- host: dashboard.com
		http:
			paths:
			- backend:
				serviceName: kubernetes-dashboard
				servicePort: 80
```

You will need to define a mapping between the host and an IP address.

### Ingress Default Backend
Ingress has a default backend. Whenever a request comes into the cluster that isn't mapped to any backend, the default backend is used to handle the request. 

This can be used for custom error messages when a request can't be handled. You just need to create an internal service with the same name `default-http-backend` and port `80` and a pod that sends that custom errorr message response.

### Ingress Use Cases
#### Multiple paths for the same host
Ingress lets you define multiple paths for the same host.

```YAML
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
	name: simple-fanout-example
	annotations:
		nginx.ingress.kubernetes.io/rewrite-target: /
spec:
	rules:
	- host: myapp.com
		http:
			paths:
			- path: /analytics
				backend:
					serviceName: analytics-service
					servicePort: 3000
			- path: /shop
				backend:
					serviceName: shopping-service
					servicePort: 8080
```

#### Multiple subdomains
Ingress can also support multiple subdomains on the same host.

```YAML
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
	name: virtual-host-ingress
spec:
	rules:
	- host: analytics.myapp.com
		http:
			paths:
				backend:
					serviceName: analytics-service
					servicePort: 3000
	- host: shop.myapp.com
		http:
			paths:
				backend:
					serviceName: shopping-service
					servicePort: 8080
```

### Configuring TLS Certificate - https
To configure a TLS certificate with Ingress, you need to specify `tls` and `secretName`, which is a reference of a Secret in a cluster that holds that tls certificate.

```YAML
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
	name: tls-example-ingress
spec:
	tls:
	- hosts: 
		- myapp.com
		secretName: myapp-secret-tls
	rules:
		- host: myapp.com
			http:
				paths:
				- path: /
					backend:
						serviceName: myapp-internal-service
						servicePort: 8080
```

The tls Secret would look like this:

```YAML
apiVersion: v1
kind: Secret
metadata:
	name: myapp-secret-tls
	namespace: default
data:
	tls.crt: base64 encoded cert
	tls.key: base64 encoded key
type: kubernetes.io/tls
```

1. The data keys need to be "tls.crt" and "tls.key". 
2. The values are file contents, **not** file paths/locations.
3. The Secret cmponent must be in the same namespace as the Ingress component.

## Helm
### Helm as Package Manager
Helm is a package manager for Kubernetes. It's a convenient way to package YAML files and dstribute them in public or private repositories.

For example, to deploy an Elastic Stack for logging in your cluster, you would need to create a StatefulSet, ConfigMap, Secret, K8s User with permissions, and a number of Services. Since Elastic Stack deployments tend to be standardized, it's more convenient to draw from a package of standard YAML files rather than writign them each time. 

A **Helm Chart** is a bundle of YAML files. With Helm, you can create your own Helm charts and push them to a Helm repository to make them available for others to use. Or you can download existing public Helm charts.

There are charts available for standard apps, such as databases and monitoring app.

You can search for charts from the command line with `helm search KEYWORD` or by going to [artifacthub.io/](https://artifacthub.io/).

Companies also maintain private registries of Helm charts to share internally.

### Helm as Templating Engine
Helm is also a templating engine.

For example, if you were deploying multiple microservices with deployment and service configurations that were almost the same, you'd need to create almost identical config files. 

Helm lets you define a common blueprint with placeholders for dynamic values.

```YAML
apiVersion: v1
kind: Pod
metadata:
	name: {{ .Values.name }}
spec:
	containers:
	- name: {{ .Values.container.name }}
		image: {{ .Values.container.image }}
		port: {{ .Values.container.port }}
```

Which draws its values from `values.yaml`. These values can also be set from the command line using the `--set` flag.

```
name: my-app
container:
	name: my-app-container
	image: my-app-image
	port: 9001
```

So instead of having yaml files for each microservice, you can have one and replace the values dynamically. This is especially practical for CI/CD since in your build pipeline you can replace the values on the fly.

Say you want to deploy the same application across across different environments, such as development, staging and production. You can make your own application chart that will have all the necessary yaml files for that deployment and use it to deploy the same application in different clusters and environments with one command.

A Helm chart wil typically have the following directory struture.

```
mychart/
	Chart.yaml
	values.yaml
	charts/
	templates/
	...
```

The top level directory will be the name of the chart.

`Chart.yaml` will contain meta information about the chart, such as name, version and a list of dependencies.

`values.yaml` will contain the values for the template files. These are the default values that you can override later.

The `charts` directory will have chart dependencies.

The `templates` directory stores the actual template files.

When you run `helm install CHARTNAME` the template files will be filled with the values from `values.yaml`, producing valid Kubernetes manifests that can then be deployed.

Optionally, you can also have other files, such as README and a licence.

#### Overriding default values
Default values can be overwritten. One way is to provide an alternative values file  when executing the install command. This will be merged with and override the values from values.yaml.

```
helm install --values=my-values.yaml CHARTNAME
```

Alternatively, you can define the values directly on the command line.

```
helm install --set version=2.0.0
```

It's more organized and managable to set the values from files.

### Release Management with Helm
In Helm 2, the Helm installation comes in two parts: the client (Helm CLI) and the server (Tiller).

When you run `helm install CHARTNAME`, Helm client will send the files to Tiller, which runs in a Kubernetes cluster. Tiller will then execute the request and create components from the yaml files.

Whenever you create or change a deployment, Tiller will store a copy of each configuration the client sent, for future reference. This creates a history of chart executions. So for example, if you execute `helm upgrade CHARTNAME`, the changes will be applied to the existing deployment instead of creating a new one.

If something goes wrong, you can rollback that upgrade using `helm rollback CHARTNAME`.

The downside of this is that Tiller has too much power inside the Kubernetes cluster, making it a security issue. This is one of the reasons Tiller was removed in Helm 3. 

## Kubernetes Volumes
Volumes allow you to persist data in Kubernetes. By default, Kubernetes doesn't offer data persistence. If you restart a MySQL pod, all of its data will be gone. You need to configure persistence for each application.

1. You need storage that doesn't depend on the pod lifecycle.
2. Storage must be available on all nodes, since you don't know on which one the pod will restart.
3. Storage needs to survive even if the cluster crashes.

You would also need persistent storage if you have an app that reads/writes to the filesystem.

You can configure any type of storage using a Kubernetes component called Persistent Volume. Think of a Persistent Volume like any other cluster resource, such as RAM or CPU.

A Persistent Volume is created via a YAML file where you specify the kind of `PersistentVolume` and spec, such as how much storage to make available. 

A Persistent Volume needs actual physical storage, such as the local disk in the cluster, an external nfs server, or cloud storage. The Persistent Volume acts as an interface to the actual storage that needs to be independently maintained.

You can think of storage as an external plugin to your cluster. You can have multiple storages configured for your cluster.

You can specify the physical storage in the `spec` section of the Persistent Volume's YAML config.

For example, using a NFS server:

```YAML
apiVersion: v1
kind: PersistentStorage
metadata:
	name: pv-name
spec:
	capacity:
		storage: 5Gi
	volumeMode: Filesystem
	accessModes:
		- ReadWriteOnce
	persistentVolumeReclaimPolicy: Recycle
	storageClassName: slow
	mountOptions:
		- hard
		- nfsvers=4.0
	nfs:
		path: /dir/path/on/nfs/server
		server: nfs-server-ip-address
```

Or using Gooogle Cloud storage:

```
apiVersion: v1
kind: PersistentStorage
metadata:
	name: test-volume
	labels:
		failure-domain.beta.kubernetes.tio/zone: us-central1-a__us-central1-b
spec:
	capacity:
		storage: 400Gi
	accessModes:
		- ReadWriteOnce
	gcePersistentDisk:
		pdName: my-data-disk
		fsType: ext4
```

Depending on the storage type, the `spec` attributes will differ. 

The [Kubernetes docs](https://kubernetes.io/docs/concepts/storage/volumes/) have a complete list of supported storages.

Note, persistent volumes are **not** namespaced and are accessible to the whole cluster.

#### Local vs Remote Volumes
Each volume type has its own use case.

Local volumes types violate the 2nd and 3rd requirements for data persistence:
- Being tied to 1 specific node
- Surviving cluster crashes

For these reasons, you should almost always use remote storage for database persistence.

Persistent volumes are resources that need to be there in the cluster before the pod that depends on it is created.

The Kubernetes Administrator sets up and maintains the cluster and ensures it has enough resources.

The Kubernetes user deploys the applications in the cluster, either directly or through a CI/CD pipeline.

The Kubernetes Admin provisions and configures the storage and creates persistent volume components from these storage backends.

Kubernetes users need to explicitly configure the application to use the persistent volume components. The application has to claim that storage. This is done using the Persistent Volume Claim (PVC) component.

PVCs are also created using YAML configuration.

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
	name: pvc-name
spec:
	storageClassName: manual
	volumeMode: Filesystem
	accessModes:
		- ReadWriteOnce
	resources:
		requests:
			storage: 10Gi
```

PVC claims a volume with a certain capacity. Whatever persistent volume satisfies this claim will be used. 

You need to use this claim in your pod's configuration. The pod's `volumes` attribute references the persistent volume claim.

```YAML
apiVersion: v1
kind: Pod
metadata:
	name: mypod
spec: 
	containers:
		- name: myfrontend
			image: nginx
			volumeMounts:
			- mountPath: "/var/www/html"
				name: mypd
	volumes:
		- name: mypd
			persistentVolumeClaim:
				claimName: pvc-name
```

#### Levels of Abstraction
- The pod requests the volume through the PV claim.
- The claim tries to find a volume in the cluster that satisfies the claim.
- The volume has the actual storage backend.

Claims must exist in the same namespace as the pods using the claim. 

Once the pod finds the persistent volume that matches the claim, the volume is mounted into the pod. Then that volume can be mounted into the container.

If you have multiple containers, you can decide to mount the volume in all of them or just some.

The application can now read and write to the storage. When the pod dies and a new one is created, it will have access to the same storage.

The benefit of this abstraction is that as a Kubernetes user, you don't need to care where the actual storage is.

### ConfigMap and Secret
ConfigMap and Secret are local volumes, not created via PV and PVC, and are managed by Kubernetes itself.

They both handle cases where you need a file available to your pod, either a configuration file or a certificate file.

You create a ConfigMap and/or Secret component and mount it into your pod/comntainer the same way you would create a persistent volume claim. 

You can configure different volume types in the same pod.

```YAML
spec:
	containers:
	- image: elastic:latest
		name: elastic-container
		ports:
		- containerPort: 9200
		volumeMounts:
		- name: es-persistent-storage
			mountPath: /var/lib/data
		- name: es-secret-dir
			mountPath: /var/lib/secret
		- name: es-config-dir
			mountPath: /var/lib/config
	volumes:
	- name: es-persistent-storage
		persistentVolumeClaim:
			claimName: es-pv-claim
	- name: es-secret-dir
		secret:
			secretName: es-secret
	- name: es-config-dir
		configMap:
			name: es-config-map
```

### Storage Class
To create persistent storage in Kubernetes.
1. Admins configure storage
2. Admins create opersistent volumes
3. Users claim PVs using PVC

There is a Kubernetes component to make this process more efficient and scalable: Storage Class. 

Storage Class provisions persistent volumes dynamically whenever a Persistent Volume Claim claims it. This means that provisioning volumes in a cluster can be automated.

Storage Class is created using a YAML configuration file:

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
	- name: storage-class-name
provisioner: kubernetes.io/aws-ebs
parameters:
	type: io1
	iopsPerGB: "10"
	fsType: ext4
```

StorageBackend is defined in the Storage Class component via the `provisioner` attribute. Each storage backend has its own provisioner. Those offered by Kubernetes (internal provisioners) are prefixed with `kubernetes.io`. You also configure parameters for the storage you want to request for the persistent volume.

Storage Class is another abstraction layer that abstracts the underlying storage provider as well as setting parameters for that storage.  

Storage Class is claimed by the Persistent Volume Claim using the `storageClassName` attribute, which references the Storage CLass to eb used to create a persistent volume that satisifes the claims of this PVC.

When a pod claims storage via PVC, the PVC will request storage from the Storage Class. The Storage Class will create a persistent volume that meets the needs of the claim.

## StatefulSet
StatefulSet is a component used for stateful applications which track state by saving this information in some storage. Compare this to stateless applications, which treat every request as a completely isolated interaction.

On account of their differences, stateful and stateless applications are deployed using different components in Kubernetes.

Stateless applications are deployed using Deployments, which allow you to replicate that application.

Stateful applications are deployed using StatefulSet. StatefulSet also makes it possible to replicate pods. 

Both manage pods based on a container specification. You can configure storage with both of them.

### Deployment vs StatefulSet
Replicating stateful applications is more difficult and has other requirements.

Scaling a Deployment is relatively easy. The pods are identical and interchangeable, and can be created in random order with random hashes. They will get one Service that load balances to any pod.

StatefulSets cannot be created and deleted at the same time and can't be randomly addressed. This is because the replica pods are not identical. They each have their own unique pod identity.

StatefulSet maintains a sticky identity for each pod. They're created from the same specification but they're not interchangeable, each ahs a persistemnt identifier that it maintains across any rescheduling.

When you start with a single database pod, it will be used for both reading and writing data. Once you add more database pods to scale the database, one pod becomes the pod that is written to, while any pod can be read from. The pod that is allowed to update the data is called the "master", while the others are called "slaves/workers". 

The different pods do not have access to the same physical storage, even though they use the same data. Each pod continuously synchronizes the data from the master pod.

When a new replica pod is added, it clones data from the previous pod. Once it has this data cloned, it starts continuous synchronization with master.

Temporary storage for a stateful aplication is possible, without persisting the data to physical storage. But this will be lost when the pods die/restart. 

If all the database pods are wiped out, the persistent physical storage will remain since persistent volume lifecycle isn't tied to other components' lifecycles.

Each pod has its own state. When a pod dies, the persistent pod identifier makes sure that the persistent volume gets reattached to the replacement pod. For this reattachment to work, it's important to use remote storage, because a pod can be rescheduled from one node to another node. Local volume storage is tied to a specific node.

With StatefulSet, every pod has its own identifier. Unlike Deployments, which get random hashes. StatefulSets get fixed ordered names in the format `STATEFFULSET_NAME-ORDINAL`, such as `mysql-0`.

If you create a StatefulSet with three replicas, you will get pods `NAME-0`, `NAME-1`, `NAME-2`. The first one is the "master", and then the "slaves" in the order of startup.

The next pod in the replica is only created once the previous pod is up and running.

Pods are deleted in reverse order starting from the last pod and waiting for each pod to be deleted before moving to the next one.

Each pod in a StatefulSet gets its own DNS endpoint from a Service. Just as with a Deployment, there is a loadbalancer service with a unique name and individual service names per pod in the format `PODNAME.GOVERNING_SERVICE_DOMAIN`, such as `mysql-0.svc2`.

Since stateful pods have a predictable pod name and a fixed individual DNS name, when pods restart the IP address will change but the name and endpoint stays the same (sticky identity). This ensures that each replica pod can retain its state and role when it dies and is recreated.

Kubernetes helps you to replicate stateful apps but you still need to:
- configure the cloning and data synchronization
- make the remote storage available
- handle backups

Stateful applications are not a perfect fit for containerized environments, which better serves stateless applications.

## Kubernetes Services
In a Kubernetes cluster, each pod gets its own internal IP address. But the pods are ephemeral and are destroyed frequently. When a pod dies and gets replaced it gets a new IP address.

Services provide a stable IP address independent of the pod's lifecycle.

A Service also provides loadbalancing, directing requests to the least busy pod.

Services are an abstraction for loose coupling for communication within and outside the cluster. 

### ClusterIP Services
The most common type of Service is the ClusterIP type. Its the default type if one isn't specified.

Imagine you have a microservice container running on port 3000 in a pod, beside it is a side-car container on port 9000 that collects the logs for the microservice.

The pod will itself have an IP address from a range that is assigned to a node. 

`kubectl get pod -o wide` will give you extra information about the pods, including their IP addresses. 

A Service is essentially an abstraction layer that represents an IP address. Service will have its own IP address and port. 

In the configuration, we define a Service by its name and the DNS resolution then maps that service to an IP address. 

Once a request reaches the Service, it knows to forward that request to one of the pods that are registered as endpoints for that service.

Service knows which pod to forward to based off the `selector` attribute in its YAML configuration. The pods will have a matching `labels` attribute in their configuration.

Service knows which port to forward to based off the `targetPort` attribute. The Service will randomly pick one of the pods and forward the request. 

When you create a Service, Kubernetes creates an Endpoint object that has the same as the Service itself. Kubernetes uses this Endpoint object to keep track of which pods are members/endpoints of the Service. The endpoints get updated whenever a pod dies or is created.

`kubectl get endpoints` will display all endpoints.

The Service's port is arbitrary (you can define it yourself), while the targetPort must match the port that the container is listening on.

### Multi-port Services
If, for example, you have a microservice and a database running in the cluster, the database can have its own Service with its own IP address. The microservice can talk to the database using this Service endpoint. 

If there is another container running in the database pod, for example for monitoring metrics, with its own endpoint, Service will have to handle multiple ports (one for the database and one for metrics).

When you have multiple ports defined in a  Service, you need to name those ports.

```YAML
ports:
	- name: mongodb
		protocol: TCP
		port: 27017
		targetPort: 27017
	- name: mongodb-exporter
		protocol: TCP
		port: 9216
		targetPort: 9216	
```

### Headless Services
The Service forwards requests randomly to pods to achieve load balancing.

Imagine a client wants to communicate with one pod directly and selectively, or that the endpoint pods need to communicate with each other directly.

You might need to do this in stateful applications, since the pod replicas are not identical. For example, with a MySQL database, the master pod is the only one allowed to write to the database and worker pods must connect to the master to synchronize their data.

In these situations, the client needs to figure out the IP address of each pod.

You could make a call to the Kubernetes API Server to get a list of pods, but this would make your application very closely tied to the Kubernetes-specific API.

As an alternative solution, Kubernetes allows clients to discover pod IP addresses through DNS lookup. 

Usually, when a client performs a DNS lookup for a Service, the DNS Server returns a single IP address, the ClusterIP. But by setting the `clusterIP` field to "none" when creating a Service, the DNS server wil return the pod IP addresses instead. The client can then do a DNS lookup to get the IP address of the pods that are members of that Service, and can connect to one or more of these pods.

When you deploy a stateful application, such as a MongoDB databse, to a cluster, you will have a normal Service and a headless Service. The normal Service will do standard load balancing, while the headless Service can handle when a client needs to communicate with a pod directly, or when the pods need to talk to each other.

### Service `type` Attributes
A Service can have one of three `type` attributes:
- ClusterIP - the default, used for internal services
- NodePort - creates a service that is externally accessible on a static port (between 30000 and 32767) on each worker node in the cluster
- LoadBalancer - makes the Service accessible externally through a Cloud Provider's LoadBalancer

When you create a NodePort Service, a ClusterIP Service is also automatically created. NodePort Services are not secure since the external clients have access to the worker nodes. 
 
When you create a LoadBalancer Service, NodePort and ClusterIP services are created automatically. The Cloud Platform's LoadBalancer will route traffic to these Services.

LoadBalancer Service is an extension of the NodePort Service which is an extension of the ClusterIP Service.

`kubectl get svc` will list all Services.

In production, you would not use NodePort for external connections. It's for testing a Service. You would configure Ingress or LoadBalancer Services for production environments.