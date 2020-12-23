<img width="2487" alt="Screen Shot 2020-10-12 at 4 59 32 PM" src="https://user-images.githubusercontent.com/26195/95800603-99d95f00-0cac-11eb-968b-f3e4dde3ff8d.png">

[![License][License-Image]][License-Url]
[![Version](https://d25lcipzij17d.cloudfront.net/badge.svg?id=go&type=5&v=0.7.4)](https://github.com/nats-io/k8s/releases/tag/v0.7.4)

[License-Url]: https://www.apache.org/licenses/LICENSE-2.0
[License-Image]: https://img.shields.io/badge/License-Apache2-blue.svg

# Running NATS on K8S

In this repository you can find several examples of how to deploy NATS, NATS Streaming 
and other tools from the NATS ecosystem on Kubernetes.

- [Getting started](#getting-started-via-the-one-line-installer)
- [Helm Charts for NATS](#helm-charts-for-nats)

## Getting started with NATS using Helm

In this repo you can find the Helm 3 based [charts](https://github.com/nats-io/k8s/tree/master/helm/charts) to install NATS and NATS Streaming (STAN).

```sh
> helm repo add nats https://nats-io.github.io/k8s/helm/charts/
> helm repo update

> helm repo list
NAME          	URL 
nats          	https://nats-io.github.io/k8s/helm/charts/

> helm install my-nats nats/nats
> helm install my-stan nats/stan --set stan.nats.url=nats://my-nats:4222
```

## Quick start using the one-line installer

Another method way to quickly bootstrap a NATS is to use the following command:

```sh
curl -sSL https://nats-io.github.io/k8s/setup.sh | sh
```

*In case you don't have a Kubernetes cluster already, you can find some notes on how to create a small cluster using one of the hosted Kubernetes providers [here](docs/create-k8s-cluster.md). You can find more info about running NATS on Kubernetes in the [docs](https://docs.nats.io/nats-on-kubernetes/minimal-setup).*

This will run a `nats-setup` container with the [required policy](https://github.com/nats-io/k8s/blob/master/setup/bootstrap-policy.yml)
and deploy a NATS cluster on Kubernetes with external access, TLS and
decentralized authorization.

[![asciicast](https://asciinema.org/a/282135.svg)](https://asciinema.org/a/282135)

By default, the installer will deploy the [Prometheus Operator](https://github.com/coreos/prometheus-operator) and the
[Cert Manager](https://github.com/jetstack/cert-manager) for metrics and TLS support, and the NATS instances will
also bind the 4222 host port for external access.

You can customize the installer to install without TLS or without Auth
to have a simpler setup as follows:

```sh
# Disable TLS
curl -sSL https://nats-io.github.io/k8s/setup.sh | sh -s -- --without-tls

# Disable Auth and TLS (also disables NATS surveyor and NATS Streaming)
curl -sSL https://nats-io.github.io/k8s/setup.sh | sh -s -- --without-tls --without-auth
```

**Note**: Since [NATS Streaming](https://github.com/nats-io/nats-streaming-server)
will be running as a [leafnode](https://github.com/nats-io/docs/tree/master/leafnodes) to NATS
(under the STAN account) and that [NATS Surveyor](https://github.com/nats-io/nats-surveyor)
requires the [system account](https://docs.nats.io/nats-server/nats_admin/sys_accounts) to monitor events, 
disabling auth also means that NATS Streaming and NATS Surveyor based monitoring will be disabled.

The monitoring dashboard setup using NATS Surveyor can be accessed by using port-forward:

    kubectl port-forward deployments/nats-surveyor-grafana 3000:3000
 
Next, open the following URL in your browser:
 
    http://127.0.0.1:3000/d/nats/nats-surveyor?refresh=5s&orgId=1

![surveyor](https://user-images.githubusercontent.com/26195/69106844-79fdd480-0a24-11ea-8e0c-213f251fad90.gif)

To cleanup the results you can run:

```sh
curl -sSL https://nats-io.github.io/k8s/destroy.sh | sh
```

## License

Unless otherwise noted, the NATS source files are distributed
under the Apache Version 2.0 license found in the LICENSE file.

## Super cluster setup
3 clusters, east, central, west
K3S for each:
### first node
```
curl -sfL https://get.k3s.io | sh -

```
Get the token:
```
sudo cat /var/lib/rancher/k3s/server/node-token
```
### second and third nodes
```
curl -sfL https://get.k3s.io | K3S_URL=https://10.40.128.120:6443 K3S_TOKEN=K10d5f393ec05e4a6a6b9b70cf9b9bd8e9479d4b7614f505c01bc7b41dcbc67b43d::server:2378b84e0e33d9efed050d8110db6d48 sh -
```
### the h8s cluster should be ready
```
$ kubectl get nodes -A
NAME                                     STATUS   ROLES    AGE     VERSION
nats-k3s-central-2.novalocal   Ready    <none>   56s     v1.19.5+k3s2
nats-k3s-central-1.novalocal   Ready    master   6m16s   v1.19.5+k3s2
nats-k3s-central-3.novalocal   Ready    <none>   3s      v1.19.5+k3s2
```
### install helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

### Add nats repo
```
helm repo add nats https://nats-io.github.io/k8s/helm/charts/
helm repo update
```

### Add seurity
```
export NSC_DIR=/home/centos/k8s/setup/nsc/
kubectl create secret generic nats-sys-creds   --from-file "$NSC_DIR/nkeys/creds/KO/SYS/sys.creds"
kubectl create secret generic nats-sys-creds   --from-file "$NSC_DIR/nkeys/creds/KO/SYS/sys.creds"
kubectl create secret generic nats-test-creds  --from-file "$NSC_DIR/nkeys/creds/KO/A/test.creds"
kubectl create secret generic nats-test2-creds --from-file "$NSC_DIR/nkeys/creds/KO/B/test.creds"
kubectl create secret generic stan-creds       --from-file "$NSC_DIR/nkeys/creds/KO/STAN/stan.creds"
kubectl create configmap nats-accounts         --from-file "$NSC_DIR/config/resolver.conf"

```

### Edit setup/nats.yaml

### Add external IP label on nodes
```
kubectl edit node nats-k3s-central-3.novalocal

    nats.io/node-external-ip: 10.40.128.85


```

### Install nats with auth
```
helm install central-nats nats/nats -f k8s/setup/nats.yaml 
```

### Surveyor on the first cluster only
```
kubectl apply --validate=false --filename k8s/tools/prometheus-operator.yml 
kubectl apply --validate=false --filename k8s/tools/nats-prometheus.yml 
kubectl apply --validate=false --filename k8s/tools/nats-surveyor-grafana.yml 
kubectl apply --validate=false --filename k8s/tools/nats-surveyor.yml 
```

### Stan on each cluster
```
helm upgrade --install east-stan nats/stan -f k8s/setup/stan.yaml 
```
