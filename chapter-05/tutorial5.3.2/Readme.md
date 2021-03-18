# Canary with Argo Rollouts
page. 138   

As you can see in section 5.3.1, using a canary deployment can help detect issues early
to prevent problematic deployment but will involve many additional steps in the
deployment process. In the next tutorial, we will use Argo Rollouts to simplify the process
of canary deployment.

### NOTE 
Please refer to section 5.1.4 on how to enable ingress and install Argo
Rollouts in your Kubernetes cluster prior to this tutorial.

1. Create the ingress, production deployment, and service (blue).
2. View the application in the browser (blue).
3. Apply the manifest with the green image with 10% of the canary traffic for 60 seconds.
4. Create the canary ingress to direct 10% of the traffic to the green service.
5. View the web page again in the browser (10% green with no error).
6. Wait 60 seconds.
7. View the application again in the browser (all green).

## Step 1. Create the ingress, production deployment, and service (blue).
First, we will create the ingress controller (listing 5.8), demo-service, and blue
deployment (listing 5.12):
```
$ kubectl apply -f ingress.yaml
ingress.extensions/demo-ingress created
configmap/nginx-configuration created

$ kubectl get ingress
NAME           CLASS    HOSTS       ADDRESS          PORTS   AGE
demo-ingress   <none>   demo.info   192.168.99.153   80      44s
```
## Step 2. View the application in the browser (blue).
## Step 3. Apply the manifest with the green image with 10% of the canary traffic for 60 seconds.
```
$ kubectl apply -f canary_rollout.yaml
rollout.argoproj.io/demo created
service/demo-service created

$ kubectl get ingress
NAME           CLASS    HOSTS       ADDRESS          PORTS   AGE
demo-ingress   <none>   demo.info   192.168.99.153   80      44s
```
### NOTE 
For the initial deployment (blue), Rollout will ignore the canary setting
and perform a regular deployment.

Once you have the ingress controller, service, and deployment created and updated
/etc/hosts(C:\Windows\System32\drivers\etc\hosts) with the demo.info and the correct IP address, you can enter the URL
demo.info and see the blue service running.

### NOTE 
NGINX Ingress Controller will only intercept traffic with the hostname
defined in the custom rule. Please make sure you add demo.info and its
IP address to your /etc/hosts(C:\Windows\System32\drivers\etc\hosts).
## Step 4. Create the canary ingress to direct 10% of the traffic to the green service.
Once the blue service is fully up and running, we can now update the manifest with
the green image and apply the manifest:
*inx
```
$ sed -i .bak 's/demo:blue/demo:green/g' canary_rollout.yaml
$ kubectl apply -f canary_rollout.yaml
rollout.argoproj.io/demo configured
service/demo-service unchanged
```
windows
```
$ sed -i  's/demo:blue/demo:green/g' canary_rollout.yaml
$ kubectl apply -f canary_rollout.yaml
rollout.argoproj.io/demo configured
service/demo-service unchanged
```
## Step 5. View the web page again in the browser (10% green with no error).
Once the canary starts, you should see something similar to figure 5.19 in section
5.3.1.
## Step 6. Wait 60 seconds.
After one minute, the green ReplicaSet will scale up while the blue deployment
scales down with all bars and dots going green (figure 5.20 in section 5.3.1).
## Step 7. View the application again in the browser (all green).