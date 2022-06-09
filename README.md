# kubernetes-new
What is Kubernetes?
Kubernetes is a portable, extensible, open source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

Kubernetes Components
A Kubernetes cluster consists of the components that represent the control plane and a set of machines called nodes.

The Kubernetes API
The Kubernetes API lets you query and manipulate the state of objects in Kubernetes. The core of Kubernetes' control plane is the API server and the HTTP API that it exposes. Users, the different parts of your cluster, and external components all communicate with one another through the API server.

Working with Kubernetes Objects
Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Learn about the Kubernetes object model and how to work with these objects.
Understanding Kubernetes Objects
This page explains how Kubernetes objects are represented in the Kubernetes API, and how you can express them in .yaml format.

Understanding Kubernetes objects
Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

What containerized applications are running (and on which nodes)
The resources available to those applications
The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance
A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's desired state.

To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the Kubernetes API. When you use the kubectl command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. You can also use the Kubernetes API directly in your own programs using one of the Client Libraries.

Object Spec and Status
Almost every Kubernetes object includes two nested object fields that govern the object's configuration: the object spec and the object status. For objects that have a spec, you have to set this when you create the object, providing a description of the characteristics you want the resource to have: its desired state.

The status describes the current state of the object, supplied and updated by the Kubernetes system and its components. The Kubernetes control plane continually and actively manages every object's actual state to match the desired state you supplied.

For example: in Kubernetes, a Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment spec to specify that you want three replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application--updating the status to match your spec. If any of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction--in this case, starting a replacement instance.

For more information on the object spec, status, and metadata, see the Kubernetes API Conventions.

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
The kubectl command-line tool supports several different ways to create and manage Kubernetes objects. This document provides an overview of the different approaches. Read the Kubectl book for details of managing objects by Kubectl.

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
Advantages compared to object configuration:

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

