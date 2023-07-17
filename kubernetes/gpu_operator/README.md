# Documentation for installing gpu-operator to run serverless function with GPU in K8S

In this example, because we are using NVIDIA GeForce RTX 4090 - 24GB RAM. So I need to using [Time-Slicing](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-sharing.html) with K8S.
- Create gpu-operator namespace
```bash
kubectl create namespace gpu-operator
```
We need create a `ConfigMap` with following content
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: time-slicing-config
  namespace: gpu-operator
data:
    4090-24gb: |-
        version: v1
        sharing:
          timeSlicing:
            resources:
            - name: nvidia.com/gpu
              replicas: 12
```

- Create ConfigMap resource with <br>
`kubectl apply -f kubernetes/gpu_operator/time-slicing/time-slicing-config.yml`

- Deploy gpu-operator helm chart
```bash
helm repo add nvidia https://helm.ngc.nvidia.com/nvidia

helm repo update

helm repo list
```
Enabling shared access to GPUs with the NVIDIA GPU Operator.
- During fresh install of the NVIDIA GPU Operator with time-slicing enabled.
```yaml
helm install gpu-operator nvidia/gpu-operator \
    -n nvidia \
    --set devicePlugin.config.name=time-slicing-config
```
- For dynamically enabling time-slicing with GPU Operator already installed.
```yaml
kubectl patch clusterpolicy/cluster-policy \
-n nvidia --type merge \
-p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config"}}}}'
```

## Applying the Time-Slicing Configuration
There are two methods:
- Across the cluster <br>
Install the GPU Operator by passing the time-slicing `ConfigMap` name and the default configuration.
```yaml
kubectl patch clusterpolicy/cluster-policy \
  -n nvidia --type merge \
  -p '{"spec": {"devicePlugin": {"config": {"name": "time-slicing-config", "default": "rtx-3070"}}}}'
```
- On certain nodes <br>
Label the node with the required time-slicing configuration in the `ConfigMap`.
```yaml
kubectl label node <node-name> nvidia.com/device-plugin.config=rtx-3070
```


**Example:** `kubectl label node vbpo-101429 nvidia.com/device-plugin.config=4090-24gb` (4090-24gb is defined in above `ConfigMap`)

To confirm gpu-operator install successful. Run `kubectl describe node vbpo-101429 | grep "nvidia.com/gpu:"
The output have shown below (12 instead of default value 1 because we defined time-slicing with 12 for GPU 4090)
```yaml
  nvidia.com/gpu:     12
  nvidia.com/gpu:     12
```













## References
- [Cấu hình cụm Kubernetes để sử dụng Nvidia GPU](https://viblo.asia/p/cau-hinh-cum-kubernetes-de-su-dung-nvidia-gpu-PwlVmyaEV5Z)
- [Run NVIDIA GPU Jobs](https://yunikorn.apache.org/docs/next/user_guide/workloads/run_nvidia/)