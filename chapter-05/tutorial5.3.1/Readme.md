# Canary with Deployment
page. 134   

In this tutorial, we will perform a canary deployment using native Kubernetes Deployment
and Service.
NOTE   
Please refer to section 5.1.4 on how to enable ingress and install Argo
Rollouts in your Kubernetes cluster prior to this tutorial.

1. Create the blue deployment and service (Production).
2. Create ingress to direct traffic to the blue service.
3. View the application in the browser (blue).
4. Deploy the green deployment (one Pod) and service, and wait for all Pods to be ready.
5. Create the canary ingress to direct 10% of traffic to the green service.
6. View the web page again in the browser (10% green with no error).
7. Scale up the green deployment to three Pods.
8. Update the canary ingress to send 100% of traffic to the green service.
9. Scale down the blue deployment to 0.


## Step 1. Create the blue deployment and service (Production).
We can create the production deployment by applying blue_deployment.yaml
(listing 5.4):
```
$ kubectl apply -f blue_deployment.yaml
deployment.apps/blue created
service/blue-service created
```
## Step 2. Create ingress to direct traffic to the blue service.
browser by applying the blue_ingress.yaml (listing 5.5). The kubectl get ingress
command will return the ingress controller hostname and IP address:
```
$ kubectl apply -f blue_ingress.yaml
ingress.extensions/demo-ingress created
configmap/nginx-configuration created
$ kubectl get ingress
NAME           CLASS    HOSTS       ADDRESS          PORTS   AGE
demo-ingress   <none>   demo.info   192.168.99.153   80      80s
```
### NOTE 
NGINX Ingress Controller will only intercept traffic with the hostname
defined in the custom rule. Please make sure you add demo.info and its
IP address to your /etc/hosts (C:\Windows\System32\drivers\etc\hosts).
## Step 3. View the application in the browser (blue).
Once you have the ingress controller, blue service, and deployment created and have
updated /etc/hosts(C:\Windows\System32\drivers\etc\hosts) with demo.info and the correct IP address, you can enter the URL
demo.info and see the blue service running.
## Step 4. Deploy the green deployment (one Pod) and service, and wait for all Pods to be ready.
Now we are ready to deploy the new green version. Let’s apply green_deployment.
yaml to create the green service and deployment:
```
$ kubectl apply -f green_deployment.yaml
deployment.apps/green created
service/green-service created
```
## Step 5. Create the canary ingress to direct 10% of traffic to the green service.
Next, we will create the canary_ingress so 10% of the traffic is routed to the canary
(green) service:
```
$ kubectl apply -f canary_ingress.yaml
ingress.extensions/canary-ingress configured
configmap/nginx-configuration unchanged
```
## Step 6. View the web page again in the browser (10% green with no error).
Now you can go back to the browser and monitor the green service.

The HTML page will show a mix of blue and green in both the
bubble and bar charts because 10% of the traffic is going to green Pods.
## Step 7. Scale up the green deployment to three Pods.
If you are able to see the correct result (healthy canary), you are ready to complete
the canary deployment (green service). We will then scale up the green deployment,
send all traffic to the green service, and scale down the blue deployment:   
*inx
```
$ sed -i .bak 's/replicas: 1/replicas: 3/g' green_deployment.yaml
$ kubectl apply -f green_deployment.yaml
deployment.apps/green configured
service/green-service unchanged
```
windows
```
$ sed -i  's/replicas: 1/replicas: 3/g' green_deployment.yaml
$ kubectl apply -f green_deployment.yaml
deployment.apps/green configured
service/green-service unchanged
```
## Step 8. Update the canary ingress to send 100% of traffic to the green service.
*inx
```
$ sed -i .bak 's/10/100/g' canary_ingress.yaml
$ kubectl apply -f canary_ingress.yaml
ingress.extensions/canary-ingress configured
configmap/nginx-configuration unchanged
```
windows
```
$ sed -i  's/10/100/g' canary_ingress.yaml
$ kubectl apply -f canary_ingress.yaml
ingress.extensions/canary-ingress configured
configmap/nginx-configuration unchanged
```
## Step 9. Scale down the blue deployment to 0.
*inx
```
$ sed -i .bak 's/replicas: 3/replicas: 0/g' blue_deployment.yaml
$ kubectl apply -f blue_deployment.yaml
deployment.apps/blue configured
service/blue-service unchanged
```
windows
```
$ sed -i 's/replicas: 3/replicas: 0/g' blue_deployment.yaml
$ kubectl apply -f blue_deployment.yaml
deployment.apps/blue configured
service/blue-service unchanged
```
Now you should be able to see all green bars and dots as 100% of the traffic is routed
to the green service.

### NOTE 
In true production, we will need to ensure all green Pods are up
before we can send 100% of the traffic to the canary service. Optionally, we
can incrementally increase the percentage of traffic to the green service while
the green deployment is scaling up.

Figure 5.20 If there is no error with the canary, the green deployment will
scale up, and the blue deployment will scale down. After the blue deployment
fully scales down, both the bubble and bar charts will show 100% green.
5.3.2 Canary