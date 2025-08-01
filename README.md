# Kubernetes-new commands
What is Kubernetes/m
Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services)), that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.////
Kubernetes Components//
A Kubernetes cluster consists of the components that represent the control plane and a set of machines called nodes
The Kubernetes API
The Kubernetes API lets you query and manipulate the state of objects in ...Kubernetes. The core of Kubernetes' control plane is the API server and the HTTP API that it exposes. Users, the different parts of your cluster, and external components all communicate with one another through the API server./
/
Working with Kubernetes Object..
Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Learn about the Kubernetes object model and how to work with these objects.//
Understanding Kubernetes Objects
This page explains how Kubernetes objects are represented in the Kubernetes API, and how you can express them in .yaml format./.
Understanding Kubernetes objects//
Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:/

What containerized applications are running (and on which nodes)
The resources available to those application.////::
The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance/
A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.////

To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the Kubernetes API. When you use the kubectl command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. You can also use the Kubernetes API directly in your own programs using one of the Client Libraries.

Object Spec and Status..
Almost every Kubernetes object includes two nested object fields that govern the object's configuration: the object spec and the object status. For objects that have a spec, you have to set this when you create the object, providing a description of the characteristics you want the resource to have: its desired state.

The status describes the current state of the object, supplied and updated by the Kubernetes system and its components. The Kubernetes control plane continually and actively manages every object's actual state to match the desired state you supplied.

For example: in Kubernetes, a Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment spec to specify that you want three replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application--updating the status to match your spec. If any of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction--in this case, starting a replacement instance.

For more information on the object spec, status, and metadata, see the Kubernetes API Conventions.
iii
Describing a Kubernetes object
When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name). When you use the Kubernetes API to create the object (either directly or via kubectl), that API request must include that information as JSON in the request body. Most often, you provide the information to kubectl in a .yaml file. kubectl converts the information to JSON when making the API request.

Here's an example .yaml file that shows the required fields and object spec for a Kubernetes Deployment:

application/deployment.yaml Copy application/deployment.yaml to clipboard
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
One way to create a Deployment using a .yaml file like the one above is to use the kubectl apply command in the kubectl command-line interface, passing the .yaml file as an argument. Here's an example:

kubectl apply -f https://k8s.io/examples/application/deployment.yaml
The output is similar to this:

deployment.apps/nginx-deployment created

Required Fields
In the .yaml file for the Kubernetes object you want to create, you'll need to set values for the following fields:

apiVersion - Which version of the Kubernetes API you're using to create this object
kind - What kind of object you want to create
metadata - Data that helps uniquely identify the object, including a name string, UID, and optional namespace
spec - What state you desire for the object
The precise format of the object spec is different for every Kubernetes object, and contains nested fields specific to that object. The Kubernetes API Reference can help you find the spec format for all of the objects you can create using Kubernetes.

For example, see the spec field for the Pod API reference. For each Pod, the .spec field specifies the pod and its desired state (such as the container image name for each container within that pod). Another example of an object specification is the spec field for the StatefulSet API. For StatefulSet, the .spec field specifies the StatefulSet and its desired state. Within the .spec of a StatefulSet is a template for Pod objects. That template describes Pods that the StatefulSet controller will create in order to satisfy the StatefulSet specification. Different kinds of object can also have different .status; again, the API reference pages detail the structure of that .status field, and its content for each different type of object.

Kubernetes Object Management
The kubectl command-line tool supports several different ways to create and manage Kubernetes objects. This document provides an overview of the different approaches. Read the Kubectl book for details of managing objects by Kubectl.////

Management techniques
Warning: A Kubernetes object should be managed using only one technique. Mixing and matching techniques for the same object results in undefined behavior.


Management technique	Operates on	Recommended environment	Supported writers	Learning curve
Imperative commands	Live objects	Development projects	1+	Lowest
Imperative object configuration	Individual files	Production projects	1	Moderate
Declarative object configuration	Directories of files	Production projects	1+	Highest
Imperative commands
When using imperative commands, a user operates directly on live objects in a cluster. The user provides operations to the kubectl command as arguments or flags.

This is the recommended way to get started or to run a one-off task in a cluster. Because this technique operates directly on live objects, it provides no history of previous configurations.

Examples
Run an instance of the nginx container by creating a Deployment object:

kubectl create deployment nginx --image nginx
Trade-offs
Advantages compared to object configuration:////

Commands are expressed as a single action word.
Commands require only a single step to make changes to the cluster.
Disadvantages compared to object configuration:

Commands do not integrate with change review processes.
Commands do not provide an audit trail associated with changes.
Commands do not provide a source of records except for what is live.
Commands do not provide a template for creating new objects.
Imperative object configuration
In imperative object configuration, the kubectl command specifies the operation (create, replace, etc.), optional flags and at least one file name. The file specified must contain a full definition of the object in YAML or JSON format.

See the API reference for more details on object definitions.

Warning: The imperative replace command replaces the existing spec with the newly provided one, dropping all changes to the object missing from the configuration file. This approach should not be used with resource types whose specs are updated independently of the configuration file. Services of type LoadBalancer, for example, have their externalIPs field updated independently from the configuration by the cluster.
Examples
Create the objects defined in a configuration file:

kubectl create -f nginx.yaml
Delete the objects defined in two configuration files:

kubectl delete -f nginx.yaml -f redis.yaml
Update the objects defined in a configuration file by overwriting the live configuration:

kubectl replace -f nginx.yaml
Trade-offs
Advantages compared to imperative commands:

Object configuration can be stored in a source control system such as Git.
Object configuration can integrate with processes such as reviewing changes before push and audit trails.
Object configuration provides a template for creating new objects.
Disadvantages compared to imperative commands:

Object configuration requires basic understanding of the object schema.
Object configuration requires the additional step of writing a YAML file.
Advantages compared to declarative object configuration:

Imperative object configuration behavior is simpler and easier to understand.
As of Kubernetes version 1.5, imperative object configuration is more mature.
Disadvantages compared to declarative object configuration:

Imperative object configuration works best on files, not directories.
Updates to live objects must be reflected in configuration files, or they will be lost during the next replacement.
Declarative object configuration
When using declarative object configuration, a user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files. Create, update, and delete operations are automatically detected per-object by kubectl. This enables working on directories, where different operations might be needed for different objects.

Note: Declarative object configuration retains changes made by other writers, even if the changes are not merged back to the object configuration file. This is possible by using the patch API operation to write only observed differences, instead of using the replace API operation to replace the entire object configuration.

Examples
Process all object configuration files in the configs directory, and create or patch the live objects. You can first diff to see what changes are going to be made, and then apply:

kubectl diff -f configs/
kubectl apply -f configs/
Recursively process directories:

kubectl diff -R -f configs/
kubectl apply -R -f configs/

Trade-offs
Advantages compared to imperative object configuration:

Changes made directly to live objects are retained, even if they are not merged back into the configuration files.
Declarative object configuration has better support for operating on directories and automatically detecting operation types (create, patch, delete) per-object.
Disadvantages compared to imperative object configuration:

Declarative object configuration is harder to debug and understand results when they are unexpected.
Partial updates using diffs create complex merge and patch operations.
Managing Kubernetes Objects Using Imperative Commands
Kubernetes objects can quickly be created, updated, and deleted directly using imperative commands built into the kubectl command-line tool. This document explains how those commands are organized and how to use them to manage live objects.

Before you begin
Install kubectl.

You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using minikube or you can use one of these Kubernetes playgrounds:
Trade-offs
The kubectl tool supports three kinds of object management:

Imperative commands
Imperative object configuration
Declarative object configuration
See Kubernetes Object Management for a discussion of the advantages and disadvantage of each kind of object management.

How to create objects
The kubectl tool supports verb-driven commands for creating some of the most common object types. The commands are named to be recognizable to users unfamiliar with the Kubernetes object types.

run: Create a new Pod to run a Container.
expose: Create a new Service object to load balance traffic across Pods.
autoscale: Create a new Autoscaler object to automatically horizontally scale a controller, such as a Deployment.
The kubectl tool also supports creation commands driven by object type. These commands support more object types and are more explicit about their intent, but require users to know the type of objects they intend to create.

create <objecttype> [<subtype>] <instancename>
Some objects types have subtypes that you can specify in the create command. For example, the Service object has several subtypes including ClusterIP, LoadBalancer, and NodePort. Here's an example that creates a Service with subtype NodePort:

kubectl create service nodeport <myservicename>
In the preceding example, the create service nodeport command is called a subcommand of the create service command.

You can use the -h flag to find the arguments and flags supported by a subcommand:

kubectl create service nodeport -h

How to update objects
The kubectl command supports verb-driven commands for some common update operations. These commands are named to enable users unfamiliar with Kubernetes objects to perform updates without knowing the specific fields that must be set:

scale: Horizontally scale a controller to add or remove Pods by updating the replica count of the controller.
annotate: Add or remove an annotation from an object.
label: Add or remove a label from an object.
The kubectl command also supports update commands driven by an aspect of the object. Setting this aspect may set different fields for different object types:

set <field>: Set an aspect of an object...
Note: In Kubernetes version 1.5, not every verb-driven command has an associated aspect-driven command.
The kubectl tool supports these additional ways to update a live object directly, however they require a better understanding of the Kubernetes object schema.

edit: Directly edit the raw configuration of a live object by opening its configuration in an editor.
patch: Directly modify specific fields of a live object by using a patch string. For more details on patch strings, see the patch section in API Conventions.
How to delete objects
You can use the delete command to delete an object from a cluster:

delete <type>/<name>
Note: You can use kubectl delete for both imperative commands and imperative object configuration. The difference is in the arguments passed to the command. To use kubectl delete as an imperative command, pass the object to be deleted as an argument. Here's an example that passes a Deployment object named nginx:
kubectl delete deployment/nginx
How to view an object
There are several commands for printing information about an object:

get: Prints basic information about matching objects. Use get -h to see a list of options.
describe: Prints aggregated detailed information about matching objects.
logs: Prints the stdout and stderr for a container running in a Pod.
Using set commands to modify objects before creation
There are some object fields that don't have a flag you can use in a create command. In some of those cases, you can use a combination of set and create to specify a value for the field before object creation. This is done by piping the output of the create command to the set command, and then back to the create command. Here's an example:

kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run=client | kubectl set selector --local -f - 'environment=qa' -o yaml | kubectl create -f -
The kubectl create service -o yaml --dry-run=client command creates the configuration for the Service, but prints it to stdout as YAML instead of sending it to the Kubernetes API server.
The kubectl set selector --local -f - -o yaml command reads the configuration from stdin, and writes the updated configuration to stdout as YAML.
The kubectl create -f - command creates the object using the configuration provided via stdin.
Using --edit to modify objects before creation
You can use kubectl create --edit to make arbitrary changes to an object before it is created. Here's an example:

kubectl create service clusterip my-svc --clusterip="None" -o yaml --dry-run=client > /tmp/srv.yaml
kubectl create --edit -f /tmp/srv.yaml
The kubectl create service command creates the configuration for the Service and saves it to /tmp/srv.yaml.
The kubectl create --edit command opens the configuration file for editing before it creates the object.

  Assign Memory Resources to Containers and Pods
This page shows how to assign a memory request and a memory limit to a Container. A Container is guaranteed to have as much memory as it requests, but is not allowed to use more memory than its limit.

Before you begin
You need to have a Kubernetes cluster, and the kubectl command-line tool must be configured to communicate with your cluster. It is recommended to run this tutorial on a cluster with at least two nodes that are not acting as control plane hosts. If you do not already have a cluster, you can create one by using minikube or you can use one of these Kubernetes playgrounds:

Katacoda
Play with Kubernetes
To check the version, enter kubectl version.
Each node in your cluster must have at least 300 MiB of memory.

A few of the steps on this page require you to run the metrics-server service in your cluster. If you have the metrics-server running, you can skip those steps.

If you are running Minikube, run the following command to enable the metrics-server:

minikube addons enable metrics-server
To see whether the metrics-server is running, or another provider of the resource metrics API (metrics.k8s.io), run the following command:

kubectl get apiservices
If the resource metrics API is available, the output includes a reference to metrics.k8s.io.

NAME
v1beta1.metrics.k8s.io
Create a namespace
Create a namespace so that the resources you create in this exercise are isolated from the rest of your cluster.

kubectl create namespace mem-example
Specify a memory request and a memory limit
To specify a memory request for a Container, include the resources:requests field in the Container's resource manifest. To specify a memory limit, include resources:limits.

In this exercise, you create a Pod that has one Container. The Container has a memory request of 100 MiB and a memory limit of 200 MiB. Here's the configuration file for the Pod:

pods/resource/memory-request-limit.yaml Copy pods/resource/memory-request-limit.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "100Mi"
      limits:
        memory: "200Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
The args section in the configuration file provides arguments for the Container when it starts. The "--vm-bytes", "150M" arguments tell the Container to attempt to allocate 150 MiB of memory.

Create the Pod:

kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit.yaml --namespace=mem-example
Verify that the Pod Container is running:

kubectl get pod memory-demo --namespace=mem-example
View detailed information about the Pod:

kubectl get pod memory-demo --output=yaml --namespace=mem-example
The output shows that the one Container in the Pod has a memory request of 100 MiB and a memory limit of 200 MiB.

...
resources:
  requests:
    memory: 100Mi
  limits:
    memory: 200Mi
...
Run kubectl top to fetch the metrics for the pod:

kubectl top pod memory-demo --namespace=mem-example
The output shows that the Pod is using about 162,900,000 bytes of memory, which is about 150 MiB. This is greater than the Pod's 100 MiB request, but within the Pod's 200 MiB limit.

NAME                        CPU(cores)   MEMORY(bytes)
memory-demo                 <something>  162856960
Delete your Pod:

kubectl delete pod memory-demo --namespace=mem-example

Exceed a Container's memory limit
A Container can exceed its memory request if the Node has memory available. But a Container is not allowed to use more than its memory limit. If a Container allocates more memory than its limit, the Container becomes a candidate for termination. If the Container continues to consume memory beyond its limit, the Container is terminated. If a terminated Container can be restarted, the kubelet restarts it, as with any other type of runtime failure.

In this exercise, you create a Pod that attempts to allocate more memory than its limit. Here is the configuration file for a Pod that has one Container with a memory request of 50 MiB and a memory limit of 100 MiB:

pods/resource/memory-request-limit-2.yaml Copy pods/resource/memory-request-limit-2.yaml to clipboard
apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-2
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-2-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "50Mi"
      limits:
        memory: "100Mi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "250M", "--vm-hang", "1"]
In the args section of the configuration file, you can see that the Container will attempt to allocate 250 MiB of memory, which is well above the 100 MiB limit.

Create the Pod:

kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit-2.yaml --namespace=mem-example
View detailed information about the Pod:

kubectl get pod memory-demo-2 --namespace=mem-example
At this point, the Container might be running or killed. Repeat the preceding command until the Container is killed:

NAME            READY     STATUS      RESTARTS   AGE
memory-demo-2   0/1       OOMKilled   1          24s
Get a more detailed view of the Container status:

kubectl get pod memory-demo-2 --output=yaml --namespace=mem-example
The output shows that the Container was killed because it is out of memory (OOM):

lastState:
   terminated:
     containerID: 65183c1877aaec2e8427bc95609cc52677a454b56fcb24340dbd22917c23b10f
     exitCode: 137
     finishedAt: 2017-06-20T20:52:19Z
     reason: OOMKilled
     startedAt: null
The Container in this exercise can be restarted, so the kubelet restarts it. Repeat this command several times to see that the Container is repeatedly killed and restarted:

kubectl get pod memory-demo-2 --namespace=mem-example
The output shows that the Container is killed, restarted, killed again, restarted again, and so on:

kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY     STATUS      RESTARTS   AGE
memory-demo-2   0/1       OOMKilled   1          37s

kubectl get pod memory-demo-2 --namespace=mem-example
NAME            READY     STATUS    RESTARTS   AGE
memory-demo-2   1/1       Running   2          40s

View detailed information about the Pod history:

kubectl describe pod memory-demo-2 --namespace=mem-example
The output shows that the Container starts and fails repeatedly:

... Normal  Created   Created container with id 66a3a20aa7980e61be4922780bf9d24d1a1d8b7395c09861225b0eba1b1f8511
... Warning BackOff   Back-off restarting failed container
View detailed information about your cluster's Nodes:

kubectl describe nodes
The output includes a record of the Container being killed because of an out-of-memory condition:

Warning OOMKilling Memory cgroup out of memory: Kill process 4481 (stress) score 1994 or sacrifice child
Delete your Pod:

kubectl delete pod memory-demo-2 --namespace=mem-example
Specify a memory request that is too big for your Nodes
Memory requests and limits are associated with Containers, but it is useful to think of a Pod as having a memory request and limit. The memory request for the Pod is the sum of the memory requests for all the Containers in the Pod. Likewise, the memory limit for the Pod is the sum of the limits of all the Containers in the Pod.

Pod scheduling is based on requests. A Pod is scheduled to run on a Node only if the Node has enough available memory to satisfy the Pod's memory request.

In this exercise, you create a Pod that has a memory request so big that it exceeds the capacity of any Node in your cluster. Here is the configuration file for a Pod that has one Container with a request for 1000 GiB of memory, which likely exceeds the capacity of any Node in your cluster.

apiVersion: v1
kind: Pod
metadata:
  name: memory-demo-3
  namespace: mem-example
spec:
  containers:
  - name: memory-demo-3-ctr
    image: polinux/stress
    resources:
      requests:
        memory: "1000Gi"
      limits:
        memory: "1000Gi"
    command: ["stress"]
    args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
Create the Pod:

kubectl apply -f https://k8s.io/examples/pods/resource/memory-request-limit-3.yaml --namespace=mem-example
View the Pod status:

kubectl get pod memory-demo-3 --namespace=mem-example
    
The output shows that the Pod status is PENDING. That is, the Pod is not scheduled to run on any Node, and it will remain in the PENDING state indefinitely:

kubectl get pod memory-demo-3 --namespace=mem-example
NAME            READY     STATUS    RESTARTS   AGE
memory-demo-3   0/1       Pending   0          25s
View detailed information about the Pod, including events:

kubectl describe pod memory-demo-3 --namespace=mem-example
The output shows that the Container cannot be scheduled because of insufficient memory on the Nodes:

Events:
  ...  Reason            Message
       ------            -------
  ...  FailedScheduling  No nodes are available that match all of the following predic
Kubernetes 201: How to Deploy a Kubernetes Cluster

In the last blog post, we discussed Kubenetes, what it is, how it interacts with containers, microservice architectures, and how Kubernetes solves a lot of the problems around container deployments.

We'll tackle the components that form a Kubernetes cluster, different ways to deploy Kubernetes, then finally a lab time where we install our own single node cluster and deploy our first workload!

Kubernetes Components

The parts of a Kubernetes cluster is a quite large topic that could fill multiple articles. We're only going to cover the basics to present a framework on how a cluster works. This might seem like a bit of a dry subject, but it will illustrate the different features of Kubernetes clusters and you'll hopefully see how they solve problems inherent in traditional software deployments.

This is a great opportunity to introduce the official Kubernetes documentation. Concerning any Kubernetes topic, the docs are a deep well of knowledge and should always be the first place you consult to learn more, the cluster components being a great example.
But first you need to know about kubectl, which is the Kubernetes command line tool used to run commands on your cluster. You'll use it to inspect different parts of your cluster, deploy and view running workloads, troubleshoot issues, and other tasks. We'll install kubectl a bit later, just know for now that it's the tool you'll use most to interact with Kubernetes.

Every cluster has two main parts, the control plane and the worker nodes. The worker nodes run your containers via instructions based from the control plane. The nodes run your containers in Docker or another container engine as pods. Pods are the smallest unit of deployment in Kubernetes, usually made up of a single container with some exceptions.

Control Plane Components

The first main component of the control plane is the API server. It receives commands issued to the cluster and sends responses back out, much like any API. When you run a command with kubectl, it essentially talks to the API server and outputs the response to your terminal.

Next is the scheduler. When you deploy a pod the scheduler figures out where would be best to run it, looking at the loads already running on each node and any specific scheduling rules in place to make a decision.

The controller watches several elements of the state of the cluster to take action if the intended state isn't met. It monitors nodes and takes action to respond if a node goes offline. It handles maintaining a set number of replicas for pods. It directs internal networking so that a pod is reachable no matter what node it is on. The controller is basically the secret sauce that makes the Kubernetes multi-node clustering magic happen.

Our last major control plane component is etcd. This is a key store that holds all the cluster data. Think of it as a database that stores all the configuration of everything running in your cluster.

All of these control plane components are designed to be run with redundancy, typically with multiple virtual machines configured as control planes. Under the hood they are managing this redundancy with complicated logic loops and quorums, which are topics way outside of our scope for today but fascinating to dive into.
# very important==]

So here we are basically discussing what is Kubernetes and Docker, what is the difference between them, how they work, and also discussing some points about Kubernetes vs Docker. Basically, these are not the same thing but the closely related. When you are working with Kubernetes you often be working with Docker.  

What are Containers?
Container package application software with their dependencies in order to abstract from the infrastructure it runs on. Now containers basically offer a logical packaging mechanism in which application can be abstracted from the environment in which they actually run. Now, this decoupling allows container-based applications to be deployed easily and consistently regardless the target environment is a private data center, the public cloud even a developer’s personal laptop.  

Key Feature of Kubernetes:
It has a tremendous amount of features which are as follows.

Runs everywhere: It is an open-source tool and gives you the freedom to take advantage of on-premises, Public & hybrid cloud infrastructure letting you move your workload to anywhere you want.
Automation:  For instance, Kubernetes will control for you with servable host the container how it will be launched.
Interaction: Kubernetes is able to manage more clusters at the same time. & It allows not only horizontal even vertical scaling also.
Additional services: It provides additional features as well as the management of containers, Kubernetes offers security networking & storage services.
self-monitoring: It also gives you a provision of self-monitoring as it constantly checks the health of nodes and the container itself.
Key Feature of Docker:
Easy configuration: This is one of the key features of Docker in which you can easily deploy your code in less time & effort as you can use Docker in a wide variety of environments. The requirement of the infrastructure is no longer linked with the environment of the application helping in configuring the system easier and faster.
You can use swarm: It is a clustering and scheduling tool for Docker containers, SO swarm used the Docker API as a frontend which helps us to use various tools to the controller, it also helps us to control cluster for Docker host as a single virtual host, it is a self-organizing group of engines that is used to enable, pluggable backbends.
Manages security: Docker allows us to save secrets in the swarm itself. And then choose to give services access to certain secrets. It includes some important commands to the engine like secret inspection, secretly creating, etc.
Services: Service is a list of tasks that lets us specify the state of a container inside of a cluster. Each task represents one instance of a container that should be running and swan schedules them across the nodes.
More Productivity: By easing technical configuration & rapid deployment of applications no doubt it has increased productivity, Docker not only helps to execute the application in an isolated environment but it also reduces the resources also.
Pros of Docker:
Build app only once: An application inside a container can run on a system that has Docker installed. So there is no need to build and configure apps multiple times on different platforms.
More sleep and less worry: With Docker, you test your application inside a container and ship it inside a container. This means the environment in which you test is identical to the one on which the app will run in production
Portability: Docker containers can run on any platform. It can run on any local system, Amazon ec2, Google cloud, Virtual box, etc.
Version control: Like git, Docker has a built version control system. Docker containers work just like GIT repositories, allowing you to commit changes to your Docker images and version control them.//
