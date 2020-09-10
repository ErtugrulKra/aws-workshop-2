# Hands-on Kubernetes-02 : Kubernetes Basic Operations

Purpose of the this hands-on training is to give students the knowledge of basic operations in Kubernetes cluster.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- Learn basic operations of nodes, podes, deployments, replicasets in Kubernetes

- Learn how to update and rollback deployments in Kubernetes

- Learn uses of namespace in Kubernetes

## Outline

- Part 1 - Basic Operations in Kubernetes

- Part 2 - Deployment Rolling Update and Rollback in Kubernetes

- Part 3 - Namespaces in Kubernetes

## Part 1 - Basic Operations in Kubernetes

- Launch a Kubernetes Cluster of Ubuntu 20.04 with two nodes (one master, one worker) using the [Cloudformation Template to Create Kubernetes Cluster](./cfn-template-to-create-k8s-cluster.yml). *Note: Once the master node up and running, worker node automatically joins the cluster.*

>*Note: If you have problem with kubernetes cluster, you can use this link for lesson.*
>https://www.katacoda.com/courses/kubernetes/playground

- Check if Kubernetes is running.

```bash

```

- Show the names and short names of the supported API resources as shown in the example:

|NAME|SHORTNAMES|
|----|----------|
|deployments|deploy
|events     |ev
|endpoints  |ep
|nodes      |no
|pods       |po
|services   |svc

```bash

```

- Get the documentation of `Nodes` and its fields.

```bash

```

- View the nodes in the cluster using.

```bash

```

- Get the documentation of `Podes` and its fields.

```bash

```
  
- Create yaml file named `mypod.yaml` and explain fields of it.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: mynginx
    image: nginx:1.19
    ports:
    - containerPort: 80
```

- Create a pod with `kubectl create` command.

```bash

```

- List the pods.

```bash

```

- List pods in `ps output format` with more information (such as node name).
  
```bash

```

- Show details of pod.

```bash

```

- Delete the pod.

```bash

```

- Get the documentation of `Deployments` and its fields.

```bash

```

- Create yaml file named `mydeployment.yaml` and explain fields of it.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.19
        ports:
        - containerPort: 80
```

- Create the deployment with `kubectl apply` command.
  
```bash

```

- List the deployments.

```bash

```

- List pods with more information.
  
```bash

```

- Show details of deployments.

```bash

```

- Print the logs for a container in a pod.

```bash

```

- If there is a multi-container pod, we can print logs of one container.

```bash

```

- Execute a command in a container.

```bash

```

```bash

```

- Open a bash shell in a container.

```bash

```

- Get the documentation of `ReplicaSets` and its fields.

```bash

```

- List the ReplicaSets.

```bash

```

- Show details of ReplicaSets.

```bash

```

- Delete a pod and show new pod is immediately created.

```bash

```

- Delete deployments

```bash

```

## Part 2 - Deployment Rolling Update and Rollback in Kubernetes

- Create a nginx deployment.

```bash

```

- List the `Deployment`, `ReplicaSet` and `Pods` of `mynginx` deployment using a label.

```bash

```

- Scale the deployment up to three replicas.

```bash

```

- List the `Deployment`, `ReplicaSet` and `Pods` of `mynginx` deployment using a label again and note the name of ReplicaSet.

```bash

```

- Describe deployment and note the image of the deployment. In our case, it is nginx:1.18-alpine.

```bash

```

- View previous rollout revisions.

```bash

```

- Display details with revision number, in our case, is 1. And note name of image.

```bash

```

- Upgrade image.

```bash

```

- Show the rollout history.

```bash

```

- Display details about the revisions.

```bash

```

- List the `Deployment`, `ReplicaSet` and `Pods` of `mynginx` deployment using a label and explain ReplicaSets.

```bash

```

- Rollback to `revision 1`.

```bash

```

- Show the rollout history and show that we have revision 2 and 3. Explain that original revision, which is `revision 1`, becomes `revision 3`.

```bash

```

- Try to pull up the `revision 1`, that is no longer available.

```bash

```

- List the `Deployment`, `ReplicaSet` and `Pods` of `mynginx` deployment using a label, and explain that the original ReplicaSet has been scaled up back to three and second ReplicaSet has been scaled down to zero.

```bash

```

- Delete the deployment.

```bash

```

## Part 3 - Namespaces in Kubernetes

- List the current namespaces in a cluster using and explain them. *Kubernetes supports multiple virtual clusters backed by the same physical cluster. These virtual clusters are called `namespaces`.*

```bash

```

>### default
>The default namespace for objects with no other namespace

>### kube-system
>The namespace for objects created by the Kubernetes system

>### kube-public
>This namespace is created automatically and is readable by all users (including those not authenticated). This >namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable >publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a >requirement.

>### kube-node-lease
>This namespace for the lease objects associated with each node which improves the performance of the node  heartbeats as the cluster scales.

- Create a new YAML file called `my-namespace.yaml` with the following content.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: clarus-namespace
```

- Create a namespace using the `my-namespace.yaml` file.

```bash

```

- Alternatively, you can create namespace using below command:

```bash

```

- Create pods in each namespace.

```bash

```

- List the deployments in `default` namespace.

```bash

```

- List the deployments in `clarus-namespace`.

```bash

```

- List the all deployments.

```bash

```

- Delete the namespace.

```bash

```
