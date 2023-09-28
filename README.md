# Demo GPUs on OpenShift

Get Nvidia GPUs setup on OpenShift and explore platform capabilities.

This repo is intended to help setup GPU prerequisites for other demos.

Try out GPUs in OpenShift Dev Spaces via a [devfile.yaml](devfile.yaml). 
Run [jupyter notebooks](notebooks) with [pytorch](notebooks/00-test-gpu-torch.ipynb)
or [tensorflow](notebooks/00-test-gpu-tensorflow.ipynb).


## Prerequisites

- Nvidia GPU Hardware
- OpenShift 4.11+
- OpenShift Dev Spaces 3.8.0+ (w/ GPUs)
- AWS (Auto Scaling)

Red Hat Demo Platform Options

- `MLOps Demo: Data Science & Edge Practice`
- `Red Hat OpenShift Container Platform 4 Demo`

## Quickstart

```
# setup operators
scripts/bootstrap.sh

# aws gpu - load functions
. scripts/bootstrap.sh

# aws gpu - basic autoscaling
setup_aws_cluster_autoscaling

# deploy gpu test pod
oc apply -f https://raw.githubusercontent.com/NVIDIA/gpu-operator/master/tests/gpu-pod.yaml

# deploy devspaces
setup_operator_devspaces
```

Setup Time Slicing (4x)

```
oc apply -k components/operators/gpu-operator-certified/instance/overlays/time-slicing-4
```

Request / Test a GPU workload of 6 GPUs

```
oc apply -k components/demos/nvidia-gpu-verification/overlays/toleration-replicas-6

# check the number of pods
oc -n nvidia-gpu-verification get pods
```

Get GPU nodes

```
oc get nodes -l node-role.kubernetes.io/gpu

oc get nodes \
  -l node-role.kubernetes.io/gpu \
  -o jsonpath={.items[*].status.allocatable} | jq . | grep nvidia
```

## Nvidia MIG profiles

*NOTE: MIG demo currently a WIP for RHDP - there be dragons here*

Manually label nodes as GPU

```
NODE=worker1.ocp.run
  oc label node/${NODE} --overwrite "node-role.kubernetes.io/gpu="
```

Setup MIG profile

```
. scripts/bootstrap.sh

# setup MIG single
# ex: nvidia.com/gpu: 1
setup_mig_config_nvidia single all-1g.5gb
setup_mig_config_nvidia single all-2g.10gb

# setup MIG mixed
# ex: nvidia.com/mig-2g.10gb: 1
setup_mig_config_nvidia mixed all-balanced
```

Manually Pick MIG profile

```
# mode = single / mixed
MIG_CONFIG=all-1g.5gb
MIG_CONFIG=all-2g.10gb

# mode = mixed 
MIG_CONFIG=all-balanced
```

Manually apply MIG partitioning profile(s) - Mixed

```
# add profile label
oc label node --overwrite \
  -l "node-role.kubernetes.io/gpu" \
  "nvidia.com/mig.config=${MIG_CONFIG}"

# remove profile label
oc label node --overwrite \
  -l "node-role.kubernetes.io/gpu" \
  "nvidia.com/mig.config-"
```

## Links

- [Additional Notes](components/operators/gpu-operator-certified/instance/INFO.md)
- [Docs - AWS GPU Instances](https://aws.amazon.com/ec2/instance-types/#Accelerated_Computing)
- [Docs - Nvidia GPU Operator on Openshift](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/openshift/contents.html)
- [Docs - Nvidia GPU admin dashboard](https://docs.openshift.com/container-platform/4.11/monitoring/nvidia-gpu-admin-dashboard.html)
- [Docs - Multi Instance GPU (MIG) in OCP](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/openshift/mig-ocp.html)
- [Docs - Time Slicing in OCP](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/openshift/time-slicing-gpus-in-openshift.html)
- [Blog - RH Nvidia GPUs on OpenShift](https://cloud.redhat.com/blog/autoscaling-nvidia-gpus-on-red-hat-openshift)
- [Demo - bkoz GPU DevSpaces](https://github.com/bkoz/devspaces)
- [GPU Operator default config map](https://gitlab.com/nvidia/kubernetes/gpu-operator/-/blob/v23.6.1/assets/state-mig-manager/0400_configmap.yaml?ref_type=tags)
