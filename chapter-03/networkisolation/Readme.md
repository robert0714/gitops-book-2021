# Page 64 Network isolation

A critical aspect of defining environments for deploying your application is to ensure
only the intended clients can access the specific environment. By default, all Namespaces
can connect to services running in all other Namespaces. But in the case of two
different environments, such as QA and Prod, you would not want cross-talk between
those environments. Luckily, it is possible to apply a Namespace network policy that
restricts network communication between Namespaces. Let’s look at how we can
deploy an application to two different Namespaces and control access using network
policies.
We will go over the steps to deploy services in two different Namespaces. You will
also modify the network policies and observe the effects.

# Exercise overview
1 Create environment Namespaces (qa and prod).  
2 Deploy curl to the qa and prod Namespaces.  
3 Deploy NGINX to the prod Namespace.  
4 Curl NGINX from both the qa and prod Namespaces (both work).
5 Block incoming traffic to the prod Namespace from the qa Namespace.  
6 Curl NGINX from the qa Namespace (blocked).

## Step 1
First, create the Namespaces for each of your environments:
```
$ kubectl create namespace qa
namespace/qa created
$ kubectl create namespace prod
namespace/prod created
$ kubectl get namespaces
NAME              STATUS   AGE
prod              Active   32s
qa                Active   40s
```
## Step 2
Now we will create a Pod in both Namespaces from where we can run the Linux command
curl:
``` 
$ kubectl -n qa apply -f curlpod.yaml
$ kubectl -n prod apply -f curlpod.yaml
```
## Step 3
In the prod Namespace, we will run an NGINX server that will receive the curl HTTP
request:
```
$ kubectl -n prod apply -f web.yaml
```

## Step 4
By default, Pods running in a Namespace can send network traffic to other Pods running
in different Namespaces. Let’s prove this by executing a curl command from the
Pod in the qa Namespace to the NGINX Pod in the prod Namespace:
``` 
# Gets the web Pod IP address
$ kubectl describe pod web -n prod | grep -i  ip

# Gets back HTTP 200
$ kubectl -n qa exec curl-pod -- curl -I http://<web pod ip>

# Gets back HTTP 200
$ kubectl -n prod exec curl-pod -- curl -I http://<web pod ip>
```
## Step 5
Typically, you never want your qa and prod environments to have dependencies
between each other. It may be that if both instances of the application were properly
configured, there would not be dependencies between qa and prod, but what if there
was a bug in the configuration of qa where it was accidentally sending traffic to prod?
You could potentially be corrupting production data. Or even within production,
what if one environment was hosting your marketing site and another environment
was hosting an HR application with sensitive data? In these cases, it may be appropriate
to block network traffic between Namespaces or only allow network traffic
between particular Namespaces. This can be accomplished by adding a ***Network-Policy*** to a Namespace.  

ps. https://kubernetes.io/docs/concepts/services-networking/network-policies/  
https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG

Let’s add a ***NetworkPolicy*** to our Pods in each Namespace:

```
$ kubectl apply -f block-other-namespace.yaml
```
CONTAINER NETWORK INTERFACE Network policy is supported only if Container 
Network Interface (CNI)2 is configured (neither minikube nor Docker desktop).
```
minikube -p gitops  start --network-plugin=cni --cni=calico
```
## Step 6
This NetworkPolicy is applied to the prod Namespace and allows only ingress
(incoming network traffic) from the prod Namespace. Correctly using ***Network-Policy*** constraints is a critical aspect of defining environment boundaries.
With the NetworkPolicy applied, we can rerun our curl commands to verify
that each Namespace is now isolated from the others:
```
# Curl from namespace qa is blocked!
$ kubectl -n qa exec curl-pod -- curl -I http://<web pod ip>

# Gets back Http 200
$ kubectl -n prod exec curl-pod -- curl -I http://<web pod ip>
```