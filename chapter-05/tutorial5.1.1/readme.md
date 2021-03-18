# Why ReplicaSet is not a good fit for GitOps
The ReplicaSet manifest includes a selector that specifies how to identify Pods it manages,
the number of replicas of Pods to maintain, and a Pod template defining how
new Pods should be created to meet the desired number of replicas. A ReplicaSet controller
will then create and delete Pods as needed to match the desired number specified
in the manifest. As we mentioned early, ReplicaSet is not declarative, and we will
use a tutorial to give you an in-depth understanding of how ReplicaSet works and why
it is not declarative:

1. Deploy a ReplicaSet with two Pods.
2. Update the image id in the manifest.
3. Apply the updated manifest and observe the ReplicaSet.
4. Update replicas to 3 in the manifest.
5. Apply the updated manifest and observe the ReplicaSet.

## Step1.  Deploy a ReplicaSet with two Pods
If ReplicaSet is declarative, you should see three Pods with the updated image id.  
First, we will apply the ReplicaSet.yaml, which will create two Pods with image id
argoproj/rollouts-demo:blue and a service:
```
kubectl apply -f ReplicaSet.yaml
```


## Step2. Update the image id in the manifest
After the deployment is completed, we will update the image id from blue to green
and apply the changes:
*inx
```
$ sed -i .bak 's/blue/green/g' ReplicaSet.yaml
$ kubectl apply -f ReplicaSet.yaml
```
windows
```
$ sed -i  's/blue/green/g' ReplicaSet.yaml
$ kubectl diff -f ReplicaSet.yaml
$ kubectl apply -f ReplicaSet.yaml
```
## Step3. Apply the updated manifest and observe the ReplicaSet
Next, we can use the kubectl diff command to verify the manifest has been
updated in Kubernetes. Then we can run kubectl get Pods and expect to see the
image tag green instead of blue:
```
$ kubectl diff -f ReplicaSet.yaml
$ kubectl get pods -o jsonpath="{.items[*].spec.containers[*].image}"
```

## Step4. Update replicas to 3 in the manifest
Even though the updated manifest has been applied, the existing Pods did not get
updated to green. Let’s update the replicas number from 2 to 3 and apply the manifest:
*inx
```
$ sed -i .bak 's/replicas: 2/replicas: 3/g' ReplicaSet.yaml
$ kubectl apply -f ReplicaSet.yaml
```
windows
```
$ sed -i  's/replicas: 2/replicas: 3/g' ReplicaSet.yaml
$ kubectl diff -f ReplicaSet.yaml
$ kubectl apply -f ReplicaSet.yaml
```
## Step5. Apply the updated manifest and observe the ReplicaSet
After applying the changes, you will see that only two Pods are running
the blue image. If ReplicaSet is declarative, all three Pods should be green.
```
$  kubectl get pods -o jsonpath="{.items[*].spec.containers[*].image}"
argoproj/rollouts-demo:green argoproj/rollouts-demo:blue argoproj/rollouts-demo:blue

$ kubectl describe rs demo
Name:         demo
Namespace:    default
Selector:     app=demo
Labels:       app=demo
Annotations:  Replicas:  3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=demo
  Containers:
   demo:
    Image:        argoproj/rollouts-demo:green
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  13m    replicaset-controller  Created pod: demo-xr6jz
  Normal  SuccessfulCreate  13m    replicaset-controller  Created pod: demo-pz4nq
  Normal  SuccessfulCreate  3m37s  replicaset-controller  Created pod: demo-hmmn2
```
Surprisingly, the third Pod’s image tag is green, but the first two Pods remain blue
because the ReplicaSet controller’s job is only to guarantee the number of running
Pods. If ReplicaSet were truly declarative, the ReplicaSet controller should detect the
changes with image tag/replicas and update all three Pods to green. In the next section,
you will see how Deployment works and why it is declarative.