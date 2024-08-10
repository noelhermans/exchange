
#### Blogs :

https://www.densify.com/kubernetes-autoscaling/kubernetes-taints/

#### Cluster with 5 worker nodes 
```shell
kubectl get nodes
NAME                                        STATUS   ROLES                       AGE     VERSION
multi-node-cx32-master1                     Ready    control-plane,etcd,master   4m1s    v1.26.4+k3s1
multi-node-cx32-pool-small-static-worker1   Ready    <none>                      3m35s   v1.26.4+k3s1
multi-node-cx32-pool-small-static-worker2   Ready    <none>                      3m35s   v1.26.4+k3s1
multi-node-cx32-pool-small-static-worker3   Ready    <none>                      3m33s   v1.26.4+k3s1
multi-node-cx32-pool-small-static-worker4   Ready    <none>                      3m33s   v1.26.4+k3s1
multi-node-cx32-pool-small-static-worker5   Ready    <none>                      3m34s   v1.26.4+k3s1
```


#### Labeling nodes 

```shell
kubectl label node multi-node-cx32-pool-small-static-worker2 size=medium
kubectl label node multi-node-cx32-pool-small-static-worker3 size=large
```
This is needed if I want to force (must) certain pods running on specific nodes.

#### Tainting nodes

```shell
kubectl taint nodes multi-node-cx32-pool-small-static-worker2 app=corr-new:NoSchedule
kubectl taint nodes multi-node-cx32-pool-small-static-worker3 app=corr-new:NoSchedule
```

#### Creating deployment

##### Without tolerations
```shell
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      run: nginx
    name: dep-std
  spec:
    progressDeadlineSeconds: 600
    replicas: 1
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        run: nginx
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        labels:
          run: nginx
      spec:
        containers:
        - image: nginx
          imagePullPolicy: Always
          name: dep-std
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
```

##### With tolerations

```shell
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      run: nginx
    name: dep-with-tol
  spec:
    progressDeadlineSeconds: 600
    replicas: 1
    revisionHistoryLimit: 10
    selector:
      matchLabels:
        run: nginx
    strategy:
      rollingUpdate:
        maxSurge: 25%
        maxUnavailable: 25%
      type: RollingUpdate
    template:
      metadata:
        labels:
          run: nginx
      spec:
        containers:
        - image: nginx
          imagePullPolicy: Always
          name: dep-with-tol
          resources: {}
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
        dnsPolicy: ClusterFirst
        restartPolicy: Always
        schedulerName: default-scheduler
        securityContext: {}
        terminationGracePeriodSeconds: 30
        tolerations:
        - effect: NoSchedule
          key: app
          operator: Equal
          value: corr-new
```

##### Results of this first test

1. DEPLOY without tolerations:  schedule on any node except worker2/3 (good)
2. DEPLOY with tolerations: schedule on any node oin the cluster (not good, we want only on worker2/3)

##### Solution for this challenge

- Statement :  A _taint_ allows a node to refuse a pod to be scheduled unless that pod has a matching _toleration_.

- If you want ensure the pods are scheduled to only those tainted nodes, also add a label to the same set of nodes and add a node affinity to the pods so that the pods can only be scheduled onto nodes with that label.

Below example is the solution -> using **2.**

#### AI

You have several ways to control pod scheduling and ensure they land on the correct nodes in Kubernetes:

**1. Node Selectors (Simplest Approach)**

   * **How it works:** You add labels to nodes and specify matching labels in your pod definitions. 
   * **Use cases:** Straightforward scenarios where you need to target pods based on static node properties.
   * **Example:**

     ```yaml
     # Node labeling (apply to nodes)
     kubectl label nodes node-1 disktype=ssd
     kubectl label nodes node-2 disktype=hdd

     # Pod definition (in your deployment/pod spec)
     apiVersion: v1
     kind: Pod
     ...
     spec:
       nodeSelector:
         disktype: ssd 
     ```

     This pod will only schedule on nodes with the label `disktype: ssd`.

**2. Node Affinity (More Expressive)**

   * **How it works:** Provides more flexible matching options, including:
      * **requiredDuringSchedulingIgnoredDuringExecution:**  (Hard requirement) The pod *must* be scheduled on a node with the matching labels.
      * **preferredDuringSchedulingIgnoredDuringExecution:** (Soft preference)  The scheduler tries its best to schedule the pod on a matching node.
   * **Use cases:** When you have more complex scheduling requirements or want to influence scheduling decisions without hard requirements.
   * **Example:**

     ```yaml
     apiVersion: v1
     kind: Pod
     ...
     spec:
       affinity:
         nodeAffinity:
           requiredDuringSchedulingIgnoredDuringExecution:
             nodeSelectorTerms:
             - matchExpressions:
               - key: disktype
                 operator: In 
                 values:
                 - ssd
                 - nvme 
           preferredDuringSchedulingIgnoredDuringExecution:
             - weight: 100 
               preference:
                 matchExpressions:
                 - key: region
                   operator: In
                   values:
                   - us-west-2 
     ```

     * This pod must be scheduled on nodes with `disktype` of either `ssd` or `nvme`.
     * It prefers to be scheduled on nodes in the `us-west-2` region. -> this is probably the option we want.

**3. Taints and Tolerations (Restricting Access)**

   * **How it works:**
      * **Taints** are applied to nodes to repel certain pods.
      * **Tolerations** are added to pods to allow them to be scheduled on tainted nodes.
   * **Use cases:**  When you need to dedicate nodes to specific purposes or prevent certain pods from running on critical nodes.
   * **Example:**

     ```yaml
     # Taint a node (apply to the node)
     kubectl taint nodes node-1 env=production:NoSchedule

     # Toleration in a pod definition
     apiVersion: v1
     kind: Pod
     ...
     spec:
       tolerations:
       - key: "env"
         operator: "Equal"
         value: "production"
         effect: "NoSchedule" 
     ```

     * The node `node-1` now has a taint that prevents pods from scheduling unless they tolerate the `env=production` taint.
     * The example pod can be scheduled on `node-1` because it includes the matching toleration.

**4. Node Resource Limits (Based on Available Resources)**

   * **How it works:** You define resource requests and limits for pods (CPU, memory), and the scheduler places pods on nodes with sufficient resources.
   * **Use cases:** Ensuring that resource-intensive pods land on nodes with enough capacity.
   * **Example:**

     ```yaml
     apiVersion: v1
     kind: Pod
     ...
     spec:
       containers:
       - name: my-container
         image
