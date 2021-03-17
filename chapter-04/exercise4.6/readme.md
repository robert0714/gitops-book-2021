# Exercise 4.6
page96  
This exercise covers how you can ensure changes are applied to Kubernetes and the
deployment completes successfully. We will use frontend-deployment.yaml as our
manifest.



1. Run kubectl diff to determine if the frontend-deployment.yaml manifest is
applied to Kubernetes. exit status 1 means the manifest is not in Kubernetes:
```
$ kubectl diff -f frontend-deployment.yaml
```
2. Apply the manifest to Kubernetes:
```
$ kubectl apply -f frontend-deployment.yaml
```
3. Rerun kubectl diff, and you should see the manifest applied with exit status 0. Kubernetes will start the deployment after the manifest update:
```
$ kubectl diff -f frontend-deployment.yaml
```
4. Repeatedly run kubectl rollout status until the deployment is fully complete:
```
$ kubectl rollout status deployment.v1.apps/frontend
Waiting for deployment "frontend" rollout to finish: 0 of 3 updated replicas are available...
Waiting for deployment "frontend" rollout to finish: 1 of 3 updated replicas are available...
Waiting for deployment "frontend" rollout to finish: 2 of 3 updated replicas are available...
deployment "frontend" successfully rolled out
```
In production, you would automate this work using a script with a loop and sleep 
around the kubectl rollout status command.

Listing 4.2 DeploymentWait.sh
```sh
#!/bin/bash
RETRY=0  # Initializes the RETRY variable to 0
STATUS="kubectl rollout status deployment.v1.apps/frontend"  # Defines the kubectl rollout status command
until $STATUS || [ $RETRY -eq 120 ]; do   # Loops exit condition when kubectl rollout status is true or the RETRY variable equals 120.  This example waits up to 20 minutes (120 x 10 seconds).
    $STATUS   # Executes the kubectl rollout status command
    RETRY=$((RETRY + 1))    # Increments the RETRY variable by 1
    sleep 10    #  Sleeps for 10 seconds
done

```