# Confidential Computing for AI Inference with Intel® TDX

Deploy confidential containers with Intel® Trust Domain Extensions (Intel® TDX) to protect models and sensitive data from unauthorized access.


## Table of contents

- [Detailed description](#detailed-description)
  - [Architecture diagrams](#architecture-diagrams)
- [Requirements](#requirements)
  - [Minimum hardware requirements](#minimum-hardware-requirements)
  - [Minimum software requirements](#minimum-software-requirements)
  - [Required user permissions](#required-user-permissions)
- [Deploy](#deploy)
  - [Clone the repository](#clone-the-repository)
  - [Create the project](#create-the-project)
  - [Build and deploy the helm chart](#build-and-deploy-the-helm-chart)
- [Test](#test)
- [Delete](#delete)
- [References](#references)


## Detailed description

When a model is run on a server, its proprietary weights, sensitive data and all queries received by the users reside in unencrypted system memory. This grants any privileged software or user, such as a cluster administrator, the ability to inspect—and potentially steal—the model and its queries during runtime. Exposing data while in use represents a severe breach of privacy, security, and intellectual property.

Intel® Trust Domain Extensions, or TDX, is Intel's newest hardware-based confidential computing technology. This hardware-based Trusted Execution Environment (TEE) facilitates the deployment of trust domains (TDs), which are hardware-isolated VMs designed to enhance the protection of sensitive data and applications from unauthorized access. The memory and register state associated with the TEE are encrypted with a key specific to the TEE and accessible only to the hardware.

Intel® TDX delivers greater confidence in data integrity, confidentiality and authenticity, which enables engineers and tech professionals to create and maintain more secure systems, establishing more trust in virtualized environments.

Confidential containers secure workloads with a seamless attestation and key release flow. The Confidential Containers runtime starts a confidential VM - a Trust Domain. Inside this secure environment, the Trustee Agents take care of performing remote attestation, reaching out to the remote attester (Trustee) to verify that the environment is trustworthy and equipped with the right software and hardware features. If the attestation passes, the same agent requests the encryption keys from the Key Broker Service — a gatekeeper that ensures only verified environments get access. Every read and write to memory is encrypted, ensuring data is never exposed, even on an untrusted host.


### Architecture diagrams

![Intel TDX](docs/images/intel_tdx.png)
*Intel® TDX features trust domains to protect data and applications from unauthorized access.*

![Confidential Containers](docs/images/confidential_containers.png)
*Confidential Containers utilize attestation services to ensure the environment is trustworthy before granting access.*

![Demo Setup](docs/diagrams/arch.svg)
*Kata Containers is a container runtime which allows running Intel TDX-protected VMs, isolating the vLLM engine and model from unauthorized access.*


## Requirements

### Minimum hardware requirements 

- 8+ vCPUs, 4th Gen Intel® Xeon® Scalable Processors or newer
- 24+ GiB RAM

**Optional, depending on selected hardware platform**
- 1 GPU (NVIDIA H100, H200, B200, or equivalent)

### Minimum software requirements

- Red Hat OpenShift
- Red Hat OpenShift AI 2.16+
- OpenShift CLI (`oc`) - [Download here](https://docs.openshift.com/container-platform/latest/cli_reference/openshift_cli/getting-started-cli.html)
- Helm CLI (`helm`) - [Download here](https://helm.sh/docs/intro/install/)

Install the following from **OperatorHub/Software Catalog** on the OpenShift console. This is a one-time setup.

#### 1. OpenShift Sandboxed Containers Operator
This is required to run Kata Containers, which are used to run Intel TDX-protected VMs (Trusted Domains).

#### 2. Node Feature Discovery (NFD)
- Install **Node Feature Discovery Operator** from OperatorHub/Software Catalog into `openshift-nfd`
- Go to **NFD → Create NodeFeatureDiscovery → Accept defaults → Create**
- Verify:
```bash
oc get pods -n openshift-nfd
# Should show nfd-controller-manager, nfd-master, nfd-worker all Running
```

#### 3. NVIDIA GPU Operator (GPU only)
- Install **NVIDIA GPU Operator** from OperatorHub/Software Catalog into `nvidia-gpu-operator`
- Go to **NVIDIA GPU Operator → Create ClusterPolicy → Accept defaults → Create**
- Wait 10-20 minutes for driver compilation, then verify:
```bash
oc get pods -n nvidia-gpu-operator
# Should show all pods Running
oc describe node $(oc get nodes -o jsonpath='{.items[0].metadata.name}') | grep -A 10 "Allocatable"
# Should show nvidia.com/gpu: 1
```

#### 4. LVM Storage
- Install a blank secondary disk. Wipe it empty and acquire the persistent path i.e. /dev/disk/by-path/pci-xxxx:xx:xx.x-nvme-x
- Install **LVM Storage** from OperatorHub/Software Catalog into `openshift-storage`
- Go to **LVM Storage → Create LVMCluster → storage → deviceClasses → deviceSelector → paths**
- Add the path to the secondary disk.
- Press "Create".
- Go to StorageClass and update this disk to be the default class. All required PersistentVolumes and PersistentVolumeClaims will be based on this StorageClass. Note down the name of this StorageClass.


### Additional

- User permissions: standard user. No elevated cluster permissions required.
- Hugging Face token: [acquire a token](https://huggingface.co/settings/tokens).


## Deploy

This AI Quickstart will deploy the [RedHatAI/Llama-4-Scout-17B-16E-Instruct-quantized.w4a16](https://huggingface.co/RedHatAI/Llama-4-Scout-17B-16E-Instruct-quantized.w4a16) model with vLLM, but secured using confidential containers powered by Intel® TDX. This model was obtained by quantizing weights of [Llama-4-Scout-17B-16E-Instruct](https://huggingface.co/meta-llama/Llama-4-Scout-17B-16E-Instruct) to INT4, reducing memory and disk size requirements by approximately 75%.

### Clone the repository

```bash
git clone https://github.com/rh-ai-quickstart/confidential-ai-inference
cd confidential-ai-inference
```

### Checkout this branch
```bash
git checkout initial-commit
```

### Create the project

```bash
PROJECT="confidential-ai-inference"
oc new-project ${PROJECT}
```

### Create the security permissions
```bash
export SA="vllm-sa"
oc create sa ${SA} -n ${PROJECT}
oc adm policy add-scc-to-user privileged -z ${SA} -n ${PROJECT}
```

### Build and deploy the helm chart

```bash
export DEVICE="gpu" # options: [gpu, cpu]
export HF_TOKEN="your-huggingface-token"
export STORAGE_CLASS_NAME="your-lvm-storageclass-name"
helm install ${PROJECT} helm/ --namespace ${PROJECT} --set device=${DEVICE} --set sa=${SA} --set hfToken=${HF_TOKEN} --set storageClassName=${STORAGE_CLASS_NAME}
```


## Test

The UI is exposed via an OpenShift Route with TLS. To get the URL, run this command:
```bash
oc get route open-webui -n $PROJECT
```

Open this URL in a web browser. Enter in prompts to generate responses from the model, while being protected by Intel® TDX.


## Delete

To uninstall and delete the project. The persistent volume claim for storing the models needs to be deleted separately.
```bash
helm uninstall $PROJECT
oc delete pvc models-cache-pvc
oc delete project $PROJECT
```
