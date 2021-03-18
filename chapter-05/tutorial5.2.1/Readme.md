# Blue-green with Deployment
page. 125
In this tutorial, we will perform a blue-green deployment using native Kubernetes
Deployment and Service.

NOTE   
Please refer to section 5.1.4 on how to enable ingress and install Argo
Rollouts in your Kubernetes cluster prior to this tutorial.

1. Create a blue deployment and service.
2. Create ingress to direct traffic to the blue service.
3. View the application in the browser (blue).
4. Deploy a green deployment and service, and wait for all Pods to be ready.
5. Update ingress to direct traffic to the green service.
6. View the web page again in the browser (green).

## Step 1. Create a blue deployment and service.
We will first create the blue deployment by applying the blue_deployment.yaml:
```
$ kubectl apply -f blue_deployment.yaml
deployment.apps/blue created
service/blue-service created
```
## Step 2. Create ingress to direct traffic to the blue service.
Now we can expose an ingress controller, so the blue service is accessible from your
browser by applying blue_ingress.yaml. The kubectl get ingress command will
return the ingress controller hostname and IP address:
```
$ kubectl apply -f blue_ingress.yaml
ingress.extensions/demo-ingress created
configmap/nginx-configuration created

$ kubectl get ingress
NAME           CLASS    HOSTS       ADDRESS          PORTS   AGE
demo-ingress   <none>   demo.info   192.168.99.152   80      47s
```
NOTE    
NGINX Ingress Controller will only intercept traffic with the hostname
defined in the custom rule. Please make sure you add demo.info and its
IP address to your /etc/hosts(C:\Windows\System32\drivers\etc\hosts).
## Step 3. View the application in the browser (blue).
Once you have the ingress controller, blue service, and deployment created and have
updated /etc/hosts (C:\Windows\System32\drivers\etc\hosts) with demo.info and the correct IP address, you can enter the URL
demo.info and see the blue service running.

### NOTE  
The demo app will continue to call the active service in the background
and show the latest results on the right side. Blue (darker gray in
print) is the running version, and green (lighter gray in print) is the new
version.

open url http://demo.info/ in browser.
## Step 4. Deploy a green deployment and service, and wait for all Pods to be ready.
Now we are ready to deploy the new green version. Let’s apply green_deployment.yaml to create the green service and deployment:
```
$ kubectl apply -f green_deployment.yaml
deployment.apps/green created
service/green-service created
```
## Step 5. Update ingress to direct traffic to the green service.
With the green service and deployment ready, we can now update the ingress controller
to route traffic to the green service:
```
$ kubectl apply -f green_ingress.yaml
ingress.extensions/demo-ingress configured
configmap/nginx-configuration unchanged
```
## Step 6. View the web page again in the browser (green).
If you go back to your browser, you should see the service turning green!
If you are happy with the deployment, you can either delete the blue service and
deployment or scale down the blue deployment to 0.
### Exercise 5.1
How would you scale down the blue deployment in a declarative way?
### Exercise 5.2
If you want a fast rollback, should you delete the deployment or scale it down to 0?