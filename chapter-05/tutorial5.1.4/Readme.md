# ARGO ROLLOUTS
The Argo Rollouts controller uses the Rollout custom
resource to provide additional deployment strategies such as blue-green and
canary to Kubernetes. The Rollout custom resource provides feature parity
with the deployment resource but with additional deployment strategies.

With minikube,6 you can enable NGINX ingress support simply by running the following
command:
```
$ minikube addons enable ingress
```
To install Argo Rollouts in your cluster, you need to create an argo-rollouts Namespace
and run install.yaml. For other environments, please refer to the Argo Rollouts
Getting Started guide:
```
$ kubectl create ns argo-rollouts
namespace/argo-rollouts created

$ kubectl apply -n argo-rollouts -f https://raw.githubusercontent.com/argoproj/argo-rollouts/stable/manifests/install.yaml
```