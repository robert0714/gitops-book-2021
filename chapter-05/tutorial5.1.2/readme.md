# How Deployment works with ReplicaSets
Deployment is fully declarative and perfectly complements GitOps. Deployment performs
rolling updates to deploy services with zero downtime. Let’s go through a tutorial
to examine how Deployment achieves rolling updates using multiple ReplicaSets.

## ROLLING UPDATES 
Rolling updates allow Deployments to update with zero
downtime by incrementally updating Pod instances with new ones. Rolling
updates work great if your service is stateless and backward compatible. Otherwise,
you will have to look into other deployment strategies, like blue-green,
which will be covered in section 5.1.3.

Let’s imagine a real-life scenario of how this would be applicable. Suppose you run a
payments service for small businesses to process credit cards. The service needs to be
available 24/7, and you have been running two Pods (blue) to handle the current
volume. You notice the two Pods are maxing out, so you decide to scale up to three
Pods (blue) to support the increased traffic. Next, your product manager wants to add
debit card support, so you need to deploy a version with three Pods (green) with zero
downtime:

1. Deploy two credit card (blue) Pods using Deployment.
2. Review Deployment and ReplicaSet.
3. Update replicas from 2 to 3 and apply the manifest.
4. Review Deployment and ReplicaSet.
5. Update the manifest with three credit and debit card (green) Pods.
6. Review Deployment and ReplicaSet while the three Pods become green.

In this tutorial, you will initially deploy two blue Pods. Then you will update/apply
the manifest to three replicas. Finally, you will update/apply the manifest to three green Pods.

## Step 1. Deploy two credit card (blue) Pods using Deployment.
Let’s start with creating the initial Deployment. As you can see from listing 5.2, the
YAML is practically the same as listing 5.1, with changes only to line 2 to use Deployment
instead of ReplicaSet:
```
$ kubectl apply -f deployment.yaml
deployment.apps/demo created
service/demo created
```
## Step 2. Review Deployment and ReplicaSet.
Let’s review what was created after we applied the Deployment manifest:
```
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-7b88c768db-qdsrt   1/1     Running   0          86s
demo-7b88c768db-xqqd9   1/1     Running   0          86s

$ kubectl get Deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   2/2     2            2           117s

kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
demo-7b88c768db   2         2         2       2m10s
```
Review  the detail of ReplicaSet.
```
$  kubectl describe rs demo-7b88c768db
Name:           demo-7b88c768db
Namespace:      default
Selector:       app=demo,pod-template-hash=7b88c768db
Labels:         app=demo
                pod-template-hash=7b88c768db
Annotations:    deployment.kubernetes.io/desired-replicas: 2
                deployment.kubernetes.io/max-replicas: 3
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/demo
Replicas:       2 current / 2 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=demo
           pod-template-hash=7b88c768db
  Containers:
   demo:
    Image:        argoproj/rollouts-demo:blue
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age    From                   Message
  ----    ------            ----   ----                   -------
  Normal  SuccessfulCreate  2m36s  replicaset-controller  Created pod: demo-7b88c768db-qdsrt
  Normal  SuccessfulCreate  2m36s  replicaset-controller  Created pod: demo-7b88c768db-xqqd9
```
Review  the data we need.
```
$  kubectl describe rs demo-7b88c768db  |grep -i controlled
Controlled By:  Deployment/demo

$  kubectl describe rs demo-7b88c768db  |grep -i Replicas
Replicas:       2 current / 2 desired

$  kubectl describe rs demo-7b88c768db  |grep -i image
    Image:        argoproj/rollouts-demo:blue
```
As expected, we have one Deployment and one ReplicaSet demo-7b88c768db created
and controlled by the demo deployment. ReplicaSet demo-7b88c768db manages two
replicas of Pods with the blue image.
## Step 3. Update replicas from 2 to 3 and apply the manifest.
Next, we will update the manifest with three
replicas and review the changes:

*inx
``` 
$ sed -i .bak 's/replicas: 2/replicas: 3/g' deployment.yaml
$ kubectl apply -f deployment.yaml
deployment.apps/demo configured
service/demo unchanged
```

windows
``` 
$ sed -i  's/replicas: 2/replicas: 3/g' deployment.yaml
$ kubectl apply -f deployment.yaml
deployment.apps/demo configured
service/demo unchanged
```
Review Deployment and ReplicaSet.
```
$ kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
demo-7b88c768db-27lcp   1/1     Running   0          13s
demo-7b88c768db-qdsrt   1/1     Running   0          10m
demo-7b88c768db-xqqd9   1/1     Running   0          10m

$  kubectl get Deployment
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
demo   3/3     3            3           11m

$  kubectl get rs
NAME              DESIRED   CURRENT   READY   AGE
demo-7b88c768db   3         3         3       11m

$ kubectl describe rs demo-7b88c768db   | grep -i image
    Image:        argoproj/rollouts-demo:blue
```
After the update, we should see the same Deployment and ReplicaSet that now manages
the three blue Pods. At this point, the Deployment looks precisely as is depicted
in figure 5.5.
## Step 4. Review Deployment and ReplicaSet.
Next, we will update the manifest to image green and apply the changes.
Since the image id has changed, Deployment will then create a second ReplicaSet to
deploy the green image.
### DEPLOYMENT AND REPLICASETS
A Deployment will create one ReplicaSet per
image id and set the number of replicas to the desired value in the ReplicaSet
with the matching image id. For all other ReplicaSets, Deployment will set
those ReplicaSets’ replica numbers to 0 to terminate all nonmatching image
id Pods.

Before you apply the change, you can open up a new terminal with the following command
to monitor the status of ReplicaSets. You should see one ReplicaSet (blue) with
three Pods:
```
$  kubectl get rs --watch
NAME              DESIRED   CURRENT   READY   AGE
demo-7b88c768db   3         3         3       14m

```
## Step 5. Update the manifest with three credit and debit card (green) Pods.
Go back to the original terminal, update the deployment, and apply the changes:
*inx
``` 
$ sed -i .bak 's/blue/green/g' deployment.yaml
$ kubectl apply -f deployment.yaml
deployment.apps/demo configured
service/demo unchanged
```
windows
``` 
$ sed -i  's/blue/green/g' deployment.yaml
$ kubectl apply -f deployment.yaml
deployment.apps/demo configured
service/demo unchanged
```
Now switch to the terminal, and you should see the ReplicaSet demo-8656dbfdc5
(blue) scaled down to 0 and a new ReplicaSet demo-6b574cb9dd (green) scaled up
to 3:
```
$  kubectl get rs --watch
NAME              DESIRED   CURRENT   READY   AGE
demo-7b88c768db   3         3         3       14m
demo-8544c6d67c   1         0         0       0s
demo-8544c6d67c   1         0         0       0s
demo-8544c6d67c   1         1         0       0s
demo-8544c6d67c   1         1         1       7s
demo-7b88c768db   2         3         3       16m
demo-7b88c768db   2         3         3       16m
demo-8544c6d67c   2         1         1       7s
demo-8544c6d67c   2         1         1       7s
demo-7b88c768db   2         2         2       16m
demo-8544c6d67c   2         2         1       7s
demo-8544c6d67c   2         2         2       11s
demo-7b88c768db   1         2         2       16m
demo-8544c6d67c   3         2         2       11s
demo-7b88c768db   1         2         2       16m
demo-8544c6d67c   3         2         2       11s
demo-7b88c768db   1         1         1       16m
demo-8544c6d67c   3         3         2       11s
demo-8544c6d67c   3         3         3       15s
demo-7b88c768db   0         1         1       16m
demo-7b88c768db   0         1         1       16m
demo-7b88c768db   0         0         0       16m
```
Let’s review what is happening here. Deployment uses the second ReplicaSet, demo-8544c6d67c, to bring up one green Pod and uses the first ReplicaSet, demo-7b88c768db,
to terminate one of the blue Pods, as depicted in figure 5.6. This process will
repeat until all three green Pods are created while all blue Pods are terminated.
## Step 6. Review Deployment and ReplicaSet while the three Pods become green.
While we are discussing Deployment, we should also cover two important configuration
parameters with the rolling-update strategy in Deployment: ***max unavailable***
and ***max surge***. Let’s review the default settings and what they mean according to the
Kubernetes documentation:

```
$ kubectl describe Deployment demo |grep RollingUpdateStrategy
RollingUpdateStrategy:  25% max unavailable, 25% max surge
```
Deployment ensures that only a certain number of Pods are down while they are
being updated. By default, it ensures that at least 75% of the desired number of Pods
are up (25% max unavailable).  
Deployment also ensures that only a certain number of Pods are created above the
desired number of Pods. By default, it ensures that at most 125% of the desired number
of Pods are up (25% max surge).  
Let’s see how it works in action. We will change the image id back to blue and configure
***max unavailable*** to 3 and ***max surge*** to 3:

```
$ kubectl apply -f deployment2.yaml
```
Now you can switch back to the terminal with ReplicaSet monitoring:
```
$ kubectl get rs --watch 
NAME              DESIRED   CURRENT   READY   AGE
demo-7b88c768db   3         3         3       14m
demo-8544c6d67c   1         0         0       0s
demo-8544c6d67c   1         0         0       0s
demo-8544c6d67c   1         1         0       0s
demo-8544c6d67c   1         1         1       7s
demo-7b88c768db   2         3         3       16m
demo-7b88c768db   2         3         3       16m
demo-8544c6d67c   2         1         1       7s
demo-8544c6d67c   2         1         1       7s
demo-7b88c768db   2         2         2       16m
demo-8544c6d67c   2         2         1       7s
demo-8544c6d67c   2         2         2       11s
demo-7b88c768db   1         2         2       16m
demo-8544c6d67c   3         2         2       11s
demo-7b88c768db   1         2         2       16m
demo-8544c6d67c   3         2         2       11s
demo-7b88c768db   1         1         1       16m
demo-8544c6d67c   3         3         2       11s
demo-8544c6d67c   3         3         3       15s
demo-7b88c768db   0         1         1       16m
demo-7b88c768db   0         1         1       16m
demo-7b88c768db   0         0         0       16m
demo-7b88c768db   0         0         0       24m
demo-7b88c768db   3         0         0       24m
demo-8544c6d67c   0         3         3       7m59s
demo-7b88c768db   3         0         0       24m
demo-8544c6d67c   0         3         3       7m59s
demo-7b88c768db   3         3         0       24m
demo-8544c6d67c   0         0         0       7m59s
demo-7b88c768db   3         3         1       24m
demo-7b88c768db   3         3         2       24m
demo-7b88c768db   3         3         3       24m
```
As you can see from the ReplicaSet change status, ReplicaSet demo-7b88c768db
(green) immediately went to zero Pods and ReplicaSet demo-8544c6d67c (blue)
immediately went to three instead of one at a time.
 
By now, you can see that Deployment achieves deployment with zero downtime by
leveraging one ReplicaSet for blue and another ReplicaSet for green. As you learn
about other deployment strategies in the rest of the chapter, you will discover that
they are all implemented similarly using two different ReplicaSets to achieve the
desired motivation.