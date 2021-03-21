# 7.4.1 HashiCorp Vault
page 187   
Vault, by HashiCorp, is a purpose built, open source tool for storing and managing
Secrets in a secure manner. Vault provides a CLI and a UI, as well as an API for programmatic
access to the Secret data. Vault is not specific to Kubernetes and is popular
as a standalone Secret management system.

## VAULT INSTALLATION AND SETUP

There are many ways to install and run Vault. But if you are new to Vault, the recommended
and easiest way to get started is to install Vault using the official Helm chart
maintained by HashiCorp. For the purposes of simplifying our tutorial, we will be
installing Vault in dev mode, which is meant for experimentation, development, and
testing. Additionally, the command also installs the Vault Agent Sidecar Injector,
which we will cover and use in the following section:
### NOTE 
requires Helm v3.0+
```
$ helm repo add hashicorp https://helm.releases.hashicorp.com
$ helm install vault hashicorp/vault \
--set server.dev.enabled=true \
--set injector.enabled=true
```
### NON-KUBERNETES INSTALLATION 
Note that it is not necessary to run Vault in a
Kubernetes environment. Vault is a general purpose Secret management system,
useful for applications and platforms other than Kubernetes. Many
enterprises choose to run a centrally managed Vault instance for their
company, so a single Vault instance can service multiple Kubernetes clusters and virtual machines, as well as be accessed by developers and operators from
the corporate network and workstations.

The Vault CLI can be downloaded from https://www.vaultproject.io/downloads or
(for macOS) by using the brew package manager:

```
$ brew install vault
```

for windows
```
$ choco install  vault
```
Once installed, Vault can be accessed through standard port forwarding, and visiting
the UI at http://localhost:8200:

```
# Run the following from a different terminal, or background it with ‘&’
$ kubectl port-forward vault-0 8200
$ export VAULT_ADDR=http://localhost:8200
```

for windows
```
PS C:\WINDOWS\system32>  kubectl port-forward vault-0 8200
PS C:\WINDOWS\system32>  $env:VAULT_ADDR="http://127.0.0.1:8200"
```

###  In dev mode, the token is the word: root
```
$ vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                root
token_accessor       O27kGeFGhBK7LEB9rjhCaOal
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
$ vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.6.2
Storage Type    inmem
Cluster Name    vault-cluster-399f7828
Cluster ID      07342934-497e-2324-6d55-e36dcfbf4674
HA Enabled      false
```

## VAULT USE
Once Vault is installed in your cluster, it’s time to store your first Secret in Vault:
```
$ vault kv put secret/hello foo=world
Key              Value
---              -----
created_time     2021-03-19T09:02:14.351532366Z
deletion_time    n/a
destroyed        false
version          1
```
To retrieve the Secret, run the vault kv get command:
```
$ vault kv get secret/hello
====== Metadata ======
Key              Value
---              -----
created_time     2021-03-19T09:02:14.351532366Z
deletion_time    n/a
destroyed        false
version          1

=== Data ===
Key    Value
---    -----
foo    world
```
By default, vault kv get will print Secrets in a tabular format. While this format is
presented in an easy-to-read way and is great for humans, it’s not as easy to parse via
automation and to be consumed by an application. To aid in this, Vault provides some
additional ways of formatting the output and extracting specific fields of the Secret:
```
$ vault kv get -field foo secret/hello
world
$ vault kv get -format json secret/hello
{
  "request_id": "36fecd1f-dd54-a4cb-0c1c-b665da49bbd0",
  "lease_id": "",
  "lease_duration": 0,
  "renewable": false,
  "data": {
    "data": {
      "foo": "world"
    },
    "metadata": {
      "created_time": "2021-03-19T09:02:14.351532366Z",
      "deletion_time": "",
      "destroyed": false,
      "version": 1
    }
  },
  "warnings": null
}
```
This makes it easy for the Vault CLI to be used in a startup script, which might
1. Run the vault kv get command to retrieve the value of a Secret.
2. Set the Secret value as an environment variable or file.
3. Start the main application, which can now read the Secret from env var or file.

An example of such a startup script might look like the following.
```
#!/bin/sh

export VAULT_TOKEN=your-vault-token
export VAULT_ADDR=https://your-vault-address.com:8200
export HELLO_SECRET=$(vault kv get -field foo secret/hello)
./guestbook
```
To integrate this with a Kubernetes application, this startup script would be used as
the entry point to the container, replacing the normal application command with the
startup script, which starts the application after the Secret has been retrieved and set to
an environment variable.

One thing to notice about this approach is that the vault kv get command itself
needs privileges to access Vault. So for this script to work, vault kv get needs to
securely communicate with the Vault server, typically using a Vault token. Another way
of saying this is that you still need a Secret to get more Secrets. This presents a
chicken-and-egg problem, where you now need to somehow securely configure and
store the Vault secret needed to retrieve the application Secrets. The solution lies in a
Kubernetes-Vault integration, which we will cover in the next section.

##  VAULT AGENT SIDECAR INJECTOR INSTALLATION AND SETUP
Earlier in the chapter, we describe how to install Vault using the official Helm chart.
This chart also includes the Agent Sidecar Injector. The instructions are repeated 
here. Note that the examples assume your current kubectl context is pointing at the
default Namespace:

```
# NOTE: requires Helm v3.0+
$ helm repo add hashicorp https://helm.releases.hashicorp.com
$ helm install vault hashicorp/vault \
--set server.dev.enabled=true \
--set injector.enabled=true
```
### USE
When an application desires to retrieve its Secrets from Vault, the Pod spec needs to
have at a minimum the following Vault agent annotations.   

vault-agent-inject-annotations.yaml
```
annotations:
  vault.hashicorp.com/agent-inject-secret-hello.txt: secret/hello
  vault.hashicorp.com/role: app
```
Breaking this down, these annotations convey several pieces of information:
* The annotation key vault.hashicorp.com/agent-inject: "true"
informs the Vault Agent Sidecar Injector that Vault Secret injection should
occur for this Pod.
* The annotation value secret/hello indicates which Vault Secret key to inject
into the Pod.
* The suffix hello.txt of the annotation, vault.hashicorp.com/agentinject-
secret-hello.txt, indicates that the Secret should be populated
under a file named hello.txt in the shared memory volume with the final path
being /vault/secrets/hello.txt.
* The annotation value from the vault.hashicorp.com/role indicates which
Vault role should be used when retrieving the Secret.


Now let’s try with a real example. To run all the Vault commands in this tutorial, you
will need to first gain console access inside Vault. Run kubectl exec to access the
interactive console of the Vault server:
```
$ kubectl exec -it vault-0 -- /bin/sh
/ $
```
If you haven’t already, follow the earlier guide on creating your first Secret named
“hello” in Vault:
```
$ vault kv put secret/hello foo=world
Key              Value
---              -----
created_time     2021-03-21T11:36:13.847762305Z
deletion_time    n/a
destroyed        false
version          1
```
Next, we need to configure Vault to allow Kubernetes Pods to authenticate and
retrieve Secrets. To do so, run the following Vault commands to enable the Kubernetes
auth method:

```
$ vault auth enable kubernetes
Success! Enabled kubernetes auth method at: kubernetes/

$ vault write auth/kubernetes/config \
  token_reviewer_jwt="$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" \
  kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

Success! Data written to: auth/kubernetes/config
```

These two commands configure Vault to use the Kubernetes authentication method to
use the service account token, the location of the Kubernetes host, and its certificate.   
Next, we define a policy named “app,” as well as a role named “app,” which will
have read privileges to the “hello” Secret:
```
# Create a policy "app" which will have read privileges to the "secret/hello" secret
$ vault policy write app - <<EOF
path "secret/hello" {
  capabilities = ["read"]
}
EOF

# Grants a pod in the "default" namespace using the "default" service account
# privileges to read the "hello" secret
$ vault write auth/kubernetes/role/app \
  bound_service_account_names=default \
  bound_service_account_namespaces=default \
  policies=app \
  ttl=24h

```
Now it’s time to deploy a Pod that will automatically get our injected Vault Secret.
Apply the following Deployment manifest, which has the Vault annotations we
described earlier on the Pod.   

vault-agent-inject-example.yaml
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: vault-agent-inject-example
spec:
  selector:
    matchLabels:
      app: vault-agent-inject-example
  template:
    metadata:
      labels:
        app: vault-agent-inject-example
      annotations:
        vault.hashicorp.com/agent-inject: "true"
        vault.hashicorp.com/agent-inject-secret-hello.txt: secret/hello
        vault.hashicorp.com/role: app
    spec:
      containers:
      - name: debian
        image: debian:latest
        command: [sleep, infinity]
```
When the deployment is up and running, we can access the console of the Pod and
verify that the Pod does indeed have the Secret mounted in it:
```
$ kubectl apply -f vault-agent-inject-example.yaml
deployment.apps/vault-agent-inject-example created

$ kubectl exec deploy/vault-agent-inject-example -it -c debian -- bash
root@vault-agent-inject-example-5c48967c97-hgzds:/# cat /vault/secrets/
hello.txt
data: map[foo:world]
metadata: map[created_time:2020-10-14T17:58:34.5584858Z deletion_time:
destroyed:false version:1]
```

Aso you can see, using the Vault Agent Sidecar Injector is one of the easiest ways to get  Vault Secrets seamlessly into your Pods in a secure manner.