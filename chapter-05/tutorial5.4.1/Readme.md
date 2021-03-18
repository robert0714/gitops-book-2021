# Progressive delivery with Argo Rollouts
page. 140   

Kubernetes does not provide an analysis tool to determine the correctness of the new
deployment. In this tutorial, we will use Argo Rollouts to implement progressive delivery.
Argo Rollouts uses the canary strategy along with ***AnalysisTemplate*** to achieve
progressive delivery.

### NOTE
Please refer to section 5.1.4 on how to enable ingress and install Argo
Rollouts in your Kubernetes cluster prior to this tutorial.

1. Create the AnalysisTemplate.
2. Create the ingress, production deployment, and service (blue).
3. Create ingress to direct traffic to the production service.
4. View the application in the browser (blue).
5. Update and apply the manifest with the green image with the Pass template.
6. View the web page again in the browser (green).
7. Update and apply the manifest with the green image with the Fail template.
8. View the application again in the browser. Still blue!

## Step 1. Create the AnalysisTemplate.
First, we will create the ***AnalysisTemplate*** (listing 5.13) for ***Rollout*** to collect metrics
and determine the health of the Pods. For simplicity, we will create one
***AnalysisTemplate pass***, which will always return 0 (healthy), and an ***Analysis-Template fail***, which will always return 1 (unhealthy). In addition, Argo Rollouts
internally maintains multiple ReplicaSets, so there is no need for multiple services.
Next, we will create the ingress controller (listing 5.8), ***demo-service***, and blue
deployment (listing 5.14):
```
$ kubectl apply -f analysis-templates.yaml
analysistemplate.argoproj.io/pass created
analysistemplate.argoproj.io/fail created
```
## Step 2. Create the ingress, production deployment, and service (blue).
```
$ kubectl apply -f ingress.yaml
ingress.extensions/demo-ingress created
configmap/nginx-configuration created
```
## Step 3. Create ingress to direct traffic to the production service.
```
$ kubectl apply -f rollout-with-analysis.yaml
rollout.argoproj.io/demo created
service/demo-service created

$ kubectl get ingress
NAME           CLASS    HOSTS       ADDRESS          PORTS   AGE
demo-ingress   <none>   demo.info   192.168.99.153   80      45s
```
#### NOTE 
For production, AnalysisTemplate has support for Prometheus,
Wavefront, and Netflix Kayenta or can be extended for other metric stores.
## Step 4. View the application in the browser (blue).
Once you have the ingress controller, service, and deployment created and updated
/etc/hosts(C:\Windows\System32\drivers\etc\hosts) with the demo.info and the correct IP address, you can enter the URL
demo.info and see the blue service running.

#### NOTE 
For the initial deployment (blue), Rollout will ignore the Canary
setting and perform a regular deployment.
## Step 5. Update and apply the manifest with the green image with the Pass template.
Once the blue service is fully up and running, we can now update the manifest
with the green image and apply the manifest. You should see the blue progressively
turning green and completely green after 20 seconds:   
*inux
```
$ sed -i .bak 's/demo:blue/demo:green/g' rollout-with-analysis.yaml
$ kubectl apply -f rollout-with-analysis.yaml
rollout.argoproj.io/demo configured
service/demo-service unchanged
```
windows
```
$ sed -i   's/demo:blue/demo:green/g' rollout-with-analysis.yaml
$ kubectl apply -f rollout-with-analysis.yaml
rollout.argoproj.io/demo configured
service/demo-service unchanged
```
## Step 6. View the web page again in the browser (green).
Once the blue service is fully up and running, we can now update the manifest
with the green image and apply the manifest.

## Step 7. Update and apply the manifest with the green image with the Fail template.
You should see the blue progressively
turning green and completely green after 20 seconds:
*inux
```
$ sed -i .bak 's/demo:blue/demo:green/g' rollout-with-analysis.yaml
$ kubectl apply -f rollout-with-analysis.yaml
rollout.argoproj.io/demo configured
service/demo-service unchanged
```
windows
```
$ sed -i   's/demo:blue/demo:green/g' rollout-with-analysis.yaml
$ kubectl apply -f rollout-with-analysis.yaml
rollout.argoproj.io/demo configured
service/demo-service unchanged
```
Figure 5.23 As the green deployment progressively scales up, both the bubble and
bar charts will gradually become all green if there is no error.
 
## Step 8. View the application again in the browser. Still blue!

Figure 5.24 The blue deployment progressively scales up but returns errors. The
blue deployment will get scaled down during failure, and both the bubble and bar
charts return to green.

#### NOTE   
The demo rollout will be marked as aborted after the failure and will
not deploy again until the abort state is set to false.

From this tutorial, you can see that Argo Rollouts uses the canary strategy to progressively
scale up the green deployment if the ***AnalysisTemplate*** continues to report
success based on the metrics collected. If ***AnalysisTemplate*** reports failures, Argo
Rollouts will roll back by scaling down the green deployment and scaling up the blue
deployment back to its original state. See table 5.1 for a deployment comparison.