# Project Notes

> Documentation: [Kubernetes.io](https://kubernetes.io/docs/concepts/workloads/)

## Commands

```sh
kubectl apply -f .
```

```sh
kubectl get all
```

```sh
kubectl port-forward svc/fleetman-webapp 30080:80
```

```sh
kubectl rollout status deploy webapp
```

```sh
kubectl rollout undo deploy webapp
# opts: --to-revision=<revision number>
```

Get Namespaces

```sh
kubectl get ns
```

Get all from certain namespace

```sh
kubectl get all -n kube-system
```

Describe service from other namespace

```sh
kubectl describe svc kube-dns -n kube-system
```

Start interactive shell inside pod

```sh
kubectl exec -it <pod name> sh
```

Get logs on specific pod

```sh
kubectl logs <pod name>
```

Get logs and follow

```sh
kubectl logs -f <pod name>
```

Get Persistent Volumes

```sh
kubectl get pv
```

Get Persistent Volume Claims

```sh
kubectl get pvc
```

## Service Types

### Pods

Pods are like "cattle": they are ephemeral, expendable, and should be replaceable.

They are defined with at least the following code:

```yml
apiVersion: v1
kind: Pod
metadata:
    name: webapp # any name
    labels: # can be any key value pairs
        app: webapp # not required
spec:
    containers:
        - name: webapp
          image: <docker-image>
```

### Services

Services are the longer-living objects defined in a cluster. Where pods are meant to be expendable, services are not. They can define multiple containers and volumes, but usually they just stick to one container.

```yml
apiVersion: v1
kind: Service
metadata:
    name: fleetman-webapp

spec:
    selector:
        app: webapp

    ports:
        - name: http
          port: 80 # external port
          nodePort: 30080 # publicly accessible on this port

    type: NodePort # expose port to the outside network
```

### Replica Sets

Replica sets are an expension of pods, in that they are extra meta information attached to normal pod definitions to define how many pods whould exist at one time. Normal pod definitions can be adapted to become replica set definitions.

```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: webapp # can be anything, but common to use same name as pod template
spec:
    selector:
      matchLabels:
        app: webapp
    replicas: 2 # any number
    template: # pod template
        ### Below is the same info used for the old pod
        metadata:
            labels:
                app: webapp
        spec:
            containers:
                - name: webapp
                  image: <docker image>
```

### Deployments

Deployments are like more sophisticated versions of replica sets, with additional features.

In addition to creating pods, they also create replica sets to manage the pods.

When a new deployment is rolled out (when code is updated, for example), a new replica set is created with new pods. The old replica set is changed to have a desired state of 0 pods. Rolling back involves changing the old replica set to have a desired state greater than 0.

## Networking

* If 2 containers are in the same pod, they can access each other via localhost or 127.0.0.1
* Each container will be wrapped in a pod, and exposed via a service
* K8 manages its own DNS service with keyvalue pairs, key being name of service and value being dynamic ip. The service is called `kube-dns`.
* When a service tries to access another service via name, it first looks up the ip via the `kube-dns` service, then uses the ip.

### Namespaces

> A namespace is a way of partitioning resources into separate areas.

This is a way to organize pods and services into different groups, like organizing frontend and backend groups into a `front-end` namespace and `back-end` namespace.

When a namespace is not specified, a service is automatically put into the default namespace.

## Databases and Storage

Volumes in Kubernetes work different than they do in Docker. In Docker, it's enough to simpy declare a `VOLUME` block; however, in Kubernetes, volumes are declared with multiple blocks to enable scalability and separation of tasks.

When creating a database with Kubernetes, one way to do it is to create a service that holds the server responsible for the database - like Mongo or Postgres. Wherever this server stores its data is then used as the mount point to connect to an external storage volume. In the case of Mongo, its data is stored in `/data/db`, so that will be the mount path.

To create a MongoDB database with persistent storage, do the following:

1. Create a Deployment
2. Create a Service for the deployment
3. Create a PVC
4. Create a Persistent Volume

## Deploy to the Cloud

* Nodes are run with EC2
* Master nodes are responsible for running all the nodes in the system (worker nodes).
* AWS EBS: Elastic Block Storage, used for storage.

### kops vs EKS

* kops is older than EKS, and is part of the original K8 repo
* EKS is younger, and is a part of AWS. 
* EKS has become more popular over recent years, especially for projects working in AWS
* If working in kops, you're responsible for master node. In EKS, amazon manages the master node for you. You can't see or manage the master node directly like you can in kops.

#### Kops -----------

Pros:

* it's well respected and heavily used
* it's easy to use

Cons:

* you are responsible for managing the master (think notifications at 3am)
* by default, you only get a single master (you can change this config though)
* it feels like more work than EKS, it's lower level to the system so more config/customization has to be done (like C++)

#### EKS -----------

Pros:

* it has gained a lot of ground recently with popularity
* using eksctl is quite simple to use
* no management of the master node
* it integrates well with Fargate (serverless compute for containers)

Cons:

* it needs a third party tool to make it usable (eksctl)
* the GUI is very poor, cant create cluster with it (which is fine if you are mainly on terminal)
* might feel like you're tied into aws (however, it's still k8, so it's still portable.)

### Pricing

Major cost is the "control plane"

Kops - running cost

* depends on Node Type you choose
* for a m3.medium, about $600/year
* LoadBalancer is about $200/year
* In total, about $800/year
* this only applies to one master node

EKS - running cost

* a set fee for the entire instance
* about $876/year
* this gives multiple masters

## Deploying with Kops

Link to docs: <https://kops.sigs.k8s.io/getting_started/aws/>

### KopsCommands

Get an overview of nodes and control plane number

```sh
kops get ig --name ${NAME}
```

Edit instance group

```sh
kops edit ig nodes-us-east-1a --name ${NAME} # use name of ig
```

Start instance

```sh
kops update cluster ${NAME} --yes --admin=87600h # gives unlimited time for admin privleges, change this.
kops export kubecfg --admin=87600h # also change this h value
```

## Working with cluster

If need the volumes to last after cluster is deleted, make sure to change the "RECLAIM POLICY" for the pv.

Get pod info:

```sh
kubectl get pods -o wide
```
