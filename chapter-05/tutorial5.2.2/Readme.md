# Blue-green with Argo Rollouts
page. 130   

Blue-green deployment is definitely doable in production using the native Kubernetes
Deployment with additional process and automation. A better approach is to make
the entire blue-green deployment process fully automated and declarative; hence,
Argo Rollouts is born.   
Argo Rollouts introduces a new custom resource called Rollout to provide additional
deployment strategies such as blue-green, canary (section 5.3), and progressive
delivery (section 5.4) to Kubernetes. The Rollout custom resource provides feature
parity with the Deployment resource with additional deployment strategies. In the
next tutorial, you will see how simple it is to deploy blue-green with Argo Rollouts.

NOTE   
Please refer to section 5.1.4 on how to enable ingress and install Argo
Rollouts in your Kubernetes cluster prior to this tutorial.

1. Deploy the NGINX Ingress Controller.
2. Deploy the production service and (blue) deployment using Argo Rollouts.
3. Update the manifest to use the green image.
4. Apply the updated manifest to deploy the new green version.


## Step 1. Deploy the NGINX Ingress Controller.
First, we will create the ingress controller, demo-service, and blue deployment:
```
$ kubectl apply -f ingress.yaml
ingress.extensions/demo-ingress created
configmap/nginx-configuration created
```
## Step 2. Deploy the production service and (blue) deployment using Argo Rollouts.
```
$ kubectl apply -f bluegreen_rollout.yaml
rollout.argoproj.io/demo created
service/demo-service created

$ kubectl get ingress
NAME           CLASS    HOSTS       ADDRESS          PORTS   AGE
demo-ingress   <none>   demo.info   192.168.99.152   80      18s

```
### NOTE   
Argo Rollouts internally will maintain one ReplicaSet for blue and one
ReplicaSet for green. It will also ensure that the green deployment is fully
scaled before updating the service selector to send all traffic over to green.
(Hence, only one service is required in this case.) In addition, Argo Rollouts
will also wait 30 seconds for all blue traffic to complete and scale down the
blue deployment.

Once you have the ingress controller, service, and deployment created and updated
/etc/hosts(C:\Windows\System32\drivers\etc\hosts), you can enter the URL demo.info and see the blue service running.

### NOTE 
NGINX Ingress Controller will only intercept traffic with the hostname
defined in the custom rule. Please make sure you add demo.info and its
IP address to your /etc/hosts(C:\Windows\System32\drivers\etc\hosts).
## Step 3. Update the manifest to use the green image.
Now we will update the manifest to deploy the new version, green. Once you have
applied the updated manifest, you can go back to your browser and see all bars and
dots turning green (figure 5.16):  
*inx
```
$ sed -i .bak 's/demo:blue/demo:green/g' bluegreen_rollout.yaml
```
windows
```
$ sed -i   's/demo:blue/demo:green/g' bluegreen_rollout.yaml
```
## Step 4. Apply the updated manifest to deploy the new green version.

```
$ kubectl apply -f bluegreen_rollout.yaml
deployment.apps/demo configured
service/demo-service unchanged
```