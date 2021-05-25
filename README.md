Requirements
============
- Latest version of Docker for Desktop with Kubernetes enabled.
- Nginx ingress controller installed in the Docker for Desktop Kubernetes cluster.  
  Additional information available here: https://kubernetes.github.io/ingress-nginx/deploy/#docker-desktop  
  `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/cloud/deploy.yaml`
- Cert Manager Installed in the Kubernetes cluster:  
  Additional information available here: https://cert-manager.io/docs/installation/kubernetes/  
  `kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml`
- Entry for kubernetes.docker.internal in your /etc/hosts file pointing to localhost. Note that some versions of Docker For Desktop will automatically add this entry into your /etc/hosts.
    ```127.0.0.1       kubernetes.docker.internal```

Warning
=======
As these manifests do not specify a namespace, this will deploy to whatever your current namespace is. I recommend creating a test namespace and updating your Kubernetes context.

    kubectl create ns alltrails-hw
    kubectl config set-context --current --namespace=alltrails-hw

TLDR;
=====
    sudo sh -c 'echo "127.0.0.1 kubernetes.docker.internal" >>/etc/hosts'
    kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.3.1/cert-manager.yaml
    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.46.0/deploy/static/provider/cloud/deploy.yaml
    kubectl apply -f alltrails-hw/manifests

To Test This Code
=================
You can verify that everything is working just by pointing your web browser at https://kubernetes.docker.internal/nav or https://kubernetes.docker.internal/authn. The placeholder pods that respond will simply reply with what they are supposed to be. Note that because we're using a basic cert manager, the browser will give a certificate security warning.

What is happening here
======================
Inbound traffic is handled by the nginx ingress-controller. The ingress controller terminates SSL and passes the unencrypted traffic on to the backing-services called "nav" and "authn." Proper reverse-proxying to the two services is based on what path is provided: either https://kubernetes.docker.local/nav or https://kubernetes.docker.local/authn respectively. SSL (tls) certificate management is accomplished through the Cert Manager tool for Kubernetes. Finally, the cluster has rules in place that will auto-scale the nav service up as it comes under more CPU load.

What doesn't work in Docker for Desktop 
=======================================
- I implemented a horizontal pod autoscaler on the nav container but the Kubernetes Metrics Server is required for it to work. Unfortunately Metrics Server and Docker for Desktop don't play well together so despite the autoscaler code being applied, scaling won't work.
- I created a sample network policy as well but network policies require a CNI that can handle them. Caleco is capable of doing that but again, it isn't super simple to install on Docker for Desktop's kubernetes so while the code is included, it is ignored.

Improvements
============
- use client-cert authentication to secure SSL traffic between the ingress controller and the services
- assuming there is going to be stateful data, we should consider backup and recovery paths
- usually raw secrets would NOT be kept unencrypted as the CA key+cert are in this repo. SOPS is a good tool for encrypting secrets in situ.
- monitoring via prometheus or similar monitoring stacks
- These manifests don't take RBAC into account but that would be an important consideration in a production environment.

Reference
=========

## Generating CA key+crt
    openssl genrsa -out ca.key 2048
    openssl req -x509 -new -nodes -key ca.key -subj "/CN=alltrails/C=US/L=OREGON" -days 100 -out ca.crt
    cat ca.key |base64 --wrap=0>ca.key.b64
    cat ca.crt |base64 --wrap=0>ca.crt.b64
