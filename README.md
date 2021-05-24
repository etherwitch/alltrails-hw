Requirements
============
- Docker for Desktop with Kubernetes enabled : latest version
- nginx ingress controller must be installed in the kubernetes cluster.
  Additional information available here: https://kubernetes.github.io/ingress-nginx/deploy/#docker-desktop
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/cloud/deploy.yaml
- cert manager must be installed in the kubernetes cluster:
  Additional information available here: https://cert-manager.io/docs/installation/kubernetes/
    kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml
- some versions of docker-for-desktop will automatically add an entry into your /etc/hosts file for kubernetes.docker.internal. If that entry doesn't already exist in your hosts file you will need to add it.
    127.0.0.1       kubernetes.docker.internal

note: use caution, this will deploy to whatever your current namespace is.

What is happening here
======================
The nginx ingress controller is a kubernetes controller that uses nginx on the back-end to handle
inbound connections to a kubernetes cluster. In this case, the ingress controller is doing a simple
ssl-termination and reverse-proxy to the two containers within the cluster.

Improvements
============
- use client-cert authentication to secure SSL traffic between the ingress controller and the services
- assuming there is going to be stateful data, we should consider backup and recovery paths
- usually raw secrets would NOT be kept unencrypted as the CA key+cert are in this repo. SOPS is a good tool for encrypting secrets in situ.
- monitoring!

ToDo
====
- scaling
- RBAC
- nginx equiv conf file
- normalize formatting of files

Reference
=========

## ingress example from cert-manager docs
https://cert-manager.io/docs/usage/ingress/


## Generating CA key+crt
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=alltrails/C=US/L=OREGON" -days 100 -out ca.crt
cat ca.key |base64 --wrap=0>ca.key.b64
cat ca.crt |base64 --wrap=0>ca.crt.b64
