#  ENFORCE AUTOMATED CHECKS
page 159  
In addition to human judgment, pull requests allow us to incorporate an automated
manifest analysis that can help to catch security issues very early. Although the ecosystem
of Kubernetes security tools is still emerging, there are already several options
available. Two good examples are ***[kubeaudit](https://github.com/Shopify/kubeaudit)*** and ***[kubesec](https://kubesec.io/)***. Both tools are available
under the Apache license and allow scanning Kubernetes manifests to find weak security
parameters.
Because our repository is open source and hosted by GitHub, we can use a CI service
such as https://travis-ci.org or https://circleci.com for free! Let’s configure an
automated kubeaudit usage and enforce successful verification for every pull request
using https://travis-ci.org:
```
git add .travis.yml
git commit -am 'Add travis config'
git push
```
Listing 6.2 .travis.yml
```yaml
language: bash
install:
- curl -sLf -o kubeaudit.tar.gz https://github.com/Shopify/kubeaudit/releases/download/v0.12.0/kubeaudit_0.12.0_linux_amd64.tar.gz
- tar -zxvf kubeaudit.tar.gz
- chmod +x kubeaudit
script:
- ./kubeaudit nonroot -f deployment.yaml &> errors
- if [ -s errors ] ; then cat errors; exit -1; fi
```