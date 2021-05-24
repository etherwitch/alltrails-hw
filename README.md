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

To Deploy This Code
===================
    cd alltrails-hw/manifests && kubectl apply -f ./

To Test This Code
=================
##You can verify that everything is working just by pointing your web browser at https://kubernetes.docker.internal/nav or https://kubernetes.docker.internal/authn. The placeholder pods that respond will simply reply with what they are supposed to be.

##You can test the auto-scaling of the nav pod by putting load on it:

    while true; do curl -k https://kubernetes.docker.local/nav; done

in another window monitor the pods to watch the nav-pod autoscale

    watch kubectl get pods

What is happening here
======================
Inbound traffic is handled by the nginx ingress-controller. The ingress controller terminates SSL and
passes the unencrypted traffic on to the backing-services "nav" and "authn" routing to the two services
based on what path is provided: either https://kubernetes.docker.local/nav or 
https://kubernetes.docker.local/authn. SSL (tls) certificate management is accomplished through the
cert-manager tool for kubernetes. It can do much more powerful things like auto-provisioning
certificates from cloud providers. Finally, the cluster has rules in place that will auto-scale the
nav service up as it comes under more CPU load.

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
