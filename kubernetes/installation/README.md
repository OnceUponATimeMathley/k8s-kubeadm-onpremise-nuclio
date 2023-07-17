# The documentation of install necessarily library and deploy k8s resources

## Quick start ⚡
- [Install nuctl](#nuctl)
- [Install helm](#helm)
- [Deploy nuclio helm chart](#nuclio-helm-chart)
- [Deploy traefik helm chart](#traefik-helm-chart)
- [Deploy Metallb](#metallb)

## NUCTL
You have to install `nuctl` command line tool to build and deploy serverless functions. Download one of the following version in [release version](https://github.com/nuclio/nuclio/releases). For example, using wget.
```bash
wget https://github.com/nuclio/nuclio/releases/download/<version>/nuctl-<version>-linux-amd64
```
After downloading the nuclio, give it a proper permission and do a softlink.
```bash
sudo chmod +x nuctl-<version>-linux-amd64
sudo ln -sf $(pwd)/nuctl-<version>-linux-amd64 /usr/local/bin/nuctl
```
After installation completed, check `nuctl version`
```bash
Client version:
"Label: 1.11.22, Git commit: 22ff6d38b0fc898af0cb96f726baab0480c1d983, OS: linux, Arch: amd64, Go version: go1.19.10"
```

# HELM
Check your corresponding operating system and [release version](https://github.com/helm/helm/releases).Install `helm` with the following commands (Ubuntu - helm version: 3.12.0).
```bash
wget https://get.helm.sh/helm-v3.12.0-linux-amd64.tar.gz
tar -zxvf helm-v3.12.0-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
```
After installation completed, check `helm version`
```bash
version.BuildInfo{Version:"v3.12.0", GitCommit:"c9f554d75773799f72ceef38c51210f1842a1dea", GitTreeState:"clean", GoVersion:"go1.20.3"}
```

## NUCLIO HELM CHART

### Prerequisites
Before starting the set-up procedure, ensure that the following prerequisites are met:

- Your environment has a Kubernetes v1.7 or later cluster.

- You have the credentials of a Docker registry, such as Docker Hub, Azure Container Registry (ACR), or Google Container Registry (GCR).

- The Nuclio CLI (nuctl) is installed — if you wish to use the CLI to deploy Nuclio functions.

### Install Nuclio
- Create a Nuclio namespace
```bash
kubectl create namespace nuclio
```
- Create a registry secret<br>
Because Nuclio functions are images that need to be pushed and pulled to/from the registry, you need to create a secret that stores your registry credentials.<br>
If you are using docker registry: Go to AccountSettings → Secret → Create token <br>
Replace the <...> placeholders in the following commands with your username, password, and URL:
```bash
read -s mypassword
<enter your password>

kubectl create secret docker-registry registry-credentials \
    --namespace nuclio \
    --docker-username <username> \
    --docker-password $mypassword \
    --docker-server <URL> \
    --docker-email ignored@nuclio.io

unset mypassword
```
- Deploy nuclio helm chart
```bash
helm repo add nuclio https://nuclio.github.io/nuclio/charts

helm install nuclio --set registry.secretName=registry-credentials  --set global.nuclio.dashboard.nodePort=32000   nuclio/nuclio -n nuclio
```

## TRAEFIK HELM CHART
- Create traefik namespace
```bash
kubectl create namespace traefik
```
- Deloy traefik helm chart
```bash
helm repo add stable https://traefik.github.io/charts

helm install -n traefik traefik stable/traefik --values=kubernetes/installation/traefik/values.yaml
```

## METALLB
After you deploy traefik helm chart successfully, check the service in master node by `kubectl get svc -A`. If your `LoadBalancer` service is `<Pending>` in `EXTERNAL-IP` column, we need to deploy MetalLB to allocate IP for my service in k8s cluster because we are using self-host on-premise, not using Cloud Provider.
<br> <br>
To install MetalLB, see [MetalLB documentation](https://metallb.universe.tf/installation/).

### Preparation
If you’re using kube-proxy in IPVS mode, since Kubernetes v1.14.2 you have to enable strict ARP mode.

Note, you don’t need this if you’re using kube-router as service-proxy because it is enabling strict ARP by default.

You can achieve this by editing kube-proxy config in current cluster:
```bash
kubectl edit configmap kube-proxy -n kube-system
```
ans set `strictARP: true`

### Installation By Manifest
```bash
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml
```

After that, we need to create IPAddressPool and L2Advertisement resource for MetalLB to allocate these IP from pool.
```bash
kubectl apply -f kubernetes/installation/metallb/pool-1.yaml

kubectl apply -f kubernetes/installation/metallb/l2-advertisement.yml
```

In `pool-1.yaml` file
```bash
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: lb-pool-1
  namespace: metallb-system
spec:
  addresses:
  - 192.168.100.100-192.168.100.120
```
The IP range `192.168.100.100-192.168.100.120` is your custom range. Because we are using Calico in `kubeadm init` with argument `--pod-network-cidr=192.168.0.0/16`. So we can choose an arbitrary valid range belong to `192.168.0.0/16`. To check my k8s cluster range. Using `kubectl cluster-info dump | grep -m 1 cluster-cidr` <br>
After deploy MetalLB successfully. Check the service in master node to confirm `EXTERNAL-IP` is not `<PENDING>`.
### References
For more information, check the following resources
- [Install MetalLB load balancer in Kubernetes](https://www.youtube.com/watch?v=Yl8JKffmhuE)
- [EXPOSING SERVICES OUTSIDE OF KUBERNETES WITH METALLB LOAD BALANCER](https://r00t.dk/post/2022/02/19/exposing-services-kubernetes-load-balancer-bare-metal-metallb/)