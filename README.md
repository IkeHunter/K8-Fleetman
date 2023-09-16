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


