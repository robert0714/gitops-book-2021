Page 61 Namespace management
# Exercise overview
1 Create environment Namespaces (guestbook-qa and guestbook-e2e).   
2 Deploy the guestbook application to the guestbook-qa environment.  
3 Test the guestbook-qa environment.  
4 Promote the guestbook application to the guestbook-e2e environment.  
5 Test the guestbook-e2e environment.

## Step 1
First, create the guestbook-qa and guestbook-e2e Namespaces for each of your guestbook
environments:
```
$ kubectl create namespace guestbook-qa
namespace/guestbook-qa created
$ kubectl create namespace guestbook-e2e
namespace/guestbook-e2e created
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   3h13m
guestbook-e2e     Active   3h11m
guestbook-qa      Active   3h12m
kube-node-lease   Active   3h13m
kube-public       Active   3h13m
kube-system       Active   3h13m
```
## Step 2
Now you can deploy the guestbook application to the guestbook-qa environment
using the following commands:
``` 
$ kubectl apply -n guestbook-qa -f  redis-master-deployment.yaml
deployment.apps/redis-master created
$ kubectl apply -n guestbook-qa -f  redis-master-service.yaml
service/redis-master created
$ kubectl apply -n guestbook-qa -f  redis-slave-deployment.yaml
deployment.apps/redis-slave created
$ kubectl apply -n guestbook-qa -f  redis-slave-service.yaml
service/redis-slave created
$ kubectl apply -n guestbook-qa -f  frontend-deployment.yaml
deployment.apps/frontend created
$ kubectl apply -n guestbook-qa -f  frontend-service.yaml
service/frontend created
```
## Step 3
Before we proceed, let’s test that the guestbook-qa environment is working as
expected. Use the following minikube command to find the URL to the guestbook-qa
service, and then open the URL in your web browser:
```
$ minikube -p gitops  service    -n guestbook-qa  frontend 
```
In the Messages text edit of the guestbook application, type something like This is
the guestbook-qa environment and press the Submit button. Your screen should
look something like figure 3.6.
## Step 4
Now that we have the Guestbook application running in the guestbook-qa environment
and have tested that it is working correctly, let’s promote guestbook-qa to the
guestbook-e2e environment. In this case, we are going to use exactly the same YAML
as was used in the guestbook-qa environment. This is similar to how your automated
CD pipeline would work:
``` 
$ kubectl apply -n guestbook-e2e -f  redis-master-deployment.yaml
deployment.apps/redis-master created
$ kubectl apply -n guestbook-e2e -f  redis-master-service.yaml
service/redis-master created
$ kubectl apply -n guestbook-e2e -f  redis-slave-deployment.yaml
deployment.apps/redis-slave created
$ kubectl apply -n guestbook-e2e -f  redis-slave-service.yaml
service/redis-slave created
$ kubectl apply -n guestbook-e2e -f  frontend-deployment.yaml
deployment.apps/frontend created
$ kubectl apply -n guestbook-e2e -f  frontend-service.yaml
service/frontend created
```
## Step 5
Great! The Guestbook app has now been deployed to the guestbook-e2e environment.
Now let’s test that the guestbook-e2e environment is working correctly:
```
$ minikube -p gitops  service    -n guestbook-e2e  frontend 
```
Similar to what you did in the guestbook-qa environment, type something like This
is the guestbook-e2e environment, NOT the guestbook-qa environment! in
the Messages text edit and press the Submit button. Your screen should look something
like figure 3.7.