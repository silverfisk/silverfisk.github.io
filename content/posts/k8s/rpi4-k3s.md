+++ 
date = 2022-07-30T13:00:00+02:00
title = "Kubernetes with k3s on Raspberry Pi 4"
slug = "" 
tags = [ "raspberry", "pi", "k8s", "k3s" ]
categories = []
thumbnail = "/img/rpi-logo.png"
description = "How to install a low footprint kubernetes on a single node Raspberry Pi 4"
+++


Disk intensive applications such as kubernetes can run on Raspberry Pi 4, but you need to [migrate from the SD card to an SSD drive](https://linux-konsult.com/posts/rpi/rpi4-ssd/).
Trust me, you don't want to run this from your SD card. I tried...

Warning: I have noted down the steps taken, there may be some additional steps you need to take to get everything working. If so let me now through [GitHub](https://github.com/silverfisk/silverfisk.github.io/blob/main/content/posts/k8s/rpi4-k3s.md)

## Preparations


K3S networking does not work with nftables, so switch to legacy iptables:
```sh
pi@raspberrypi:~ $ sudo iptables -F
pi@raspberrypi:~ $ sudo update-alternatives --set iptables /usr/sbin/iptables-legacy
pi@raspberrypi:~ $ sudo update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
pi@raspberrypi:~ $ sudo reboot
```

The currently [latest release](https://github.com/k3s-io/k3s/releases) of k3s (v1.24.3+k3s1) is not compatible with the rancher helm chart.
If you've received a message like 
> Error: chart requires kubeVersion: < 1.24.0-0 which is incompatible with Kubernetes

Installing k3s version v1.23.9+k3s1 like below is a workaround if you want to install rancher with the helm chart.

```sh
pi@raspberrypi:~ $ sudo su -
# curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.23.9+k3s1 K3S_NODE_NAME=k8s.example.com sh -
# curl -sfL https://get.k3s.io | sh -
# kubectl get nodes
# kubectl create namespace cert-manager
# kubectl --namespace cert-manager get pods
# wget https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cmctl-linux-arm64.tar.gz
# tar xvzf cmctl-linux-arm64.tar.gz --dir=/usr/local/bin/ cmctl && rm -f cmctl-linux-arm64.tar.gz
# cmctl check api || kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.9.1/cert-manager.yaml
# while cmctl check api ; [ $? -ne 0 ];do sleep 3; done             # Loop until the cert-manager API is ready
# kubectl get pods --namespace cert-manager                         # Should show 3 pods in running state
# kubectl config view --raw > ~/.kube/config
# exit
pi@raspberrypi:~ $ sudo kubectl config view --raw > ~/.kube/config
```

Now K3S and cert-manager is installed. Lets test it by creating a dummy Issuer:

```sh
pi@raspberrypi:~ $ cat <<EOF > /tmp/test-resources.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager-test
---
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: test-selfsigned
  namespace: cert-manager-test
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: selfsigned-cert
  namespace: cert-manager-test
spec:
  dnsNames:
    - example.com
  secretName: selfsigned-cert-tls
  issuerRef:
    name: test-selfsigned
EOF
pi@raspberrypi:~ $ kubectl apply -f  /tmp/test-resources.yaml
pi@raspberrypi:~ $ kubectl describe certificate -n cert-manager-test
pi@raspberrypi:~ $ kubectl delete -f  /tmp/test-resources.yaml
```

If all steps above worked, create a LetsEncrypt certificate following [this guide](https://cert-manager.io/docs/tutorials/acme/dns-validation/#issuing-an-acme-certificate-using-dns-validation)


```sh
pi@raspberrypi:~ $ kubectl get svc --all-namespaces
pi@raspberrypi:~ $ cat <<EOF > letsencrypt-issuer-staging.yaml 
apiVersion: cert-manager.io/v1
kind: Issuer
metadata:
  name: letsencrypt-staging
  namespace: cert-manager-test
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: you@example.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging
    # Enable the HTTP-01 challenge provider
    solvers:
    # An empty 'selector' means that this solver matches all domains
    - selector: {}
      http01:
        ingress:
          class: traefik
EOF
pi@raspberrypi:~ $ kubectl get issuer -A
pi@raspberrypi:~ $ cat <<EOF > letsencrypt-issuer-staging.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-staging
spec:
  acme:
    # The ACME server URL
    server: https://acme-staging-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: you@example.com
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-staging
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: traefik
EOF
pi@raspberrypi:~ $ kubectl apply -f letsencrypt-issuer-staging.yaml
pi@raspberrypi:~ $ kubectl get clusterissuer -A
pi@raspberrypi:~ $ cat <<EOF > letsencrypt-test-certificate.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: k3s-example-com
  namespace: default
spec:
  secretName: k3s-example-com-tls
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
  commonName: k3s.example.com
  dnsNames:
  - k3s.example.com
EOF
pi@raspberrypi:~ $ 
pi@raspberrypi:~ $ 
pi@raspberrypi:~ $ kubectl delete -f  letsencrypt-issuer-staging.yaml
```
