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

Install the following operators on the OpenShift console:
- OpenShift Sandboxed Containers Operator 1.9.0+

**Required if running with GPU**

Install the following operators on the OpenShift console:
- Node Feature Discovery Operator 4.21.0+
- Kernel Module Management Operator 2.5.1+
- NVIDIA GPU Operator 25.10.1+

### Required user permissions

- Standard user. No elevated cluster permissions required.


## Deploy

This AI Quickstart will deploy a `Llama-4-Scout-17B-16E-Instruct-quantized` model with vLLM, but secured using confidential containers powered by Intel® TDX.

### Clone the repository

```bash
git clone https://github.com/rh-ai-quickstart/confidential-ai-inference
cd confidential-ai-inference
```

### Create the project

```bash
PROJECT="confidential-ai-demo"
oc new-project ${PROJECT}
```

### Create the security permissions (Required for GPU only)
```bash
export SA="vllm-sa"
oc create sa ${SA} -n ${PROJECT}
oc adm policy add-scc-to-user privileged -z ${SA} -n ${PROJECT}
```

### Build and deploy the helm chart

```bash
export DEVICE="gpu" # options: [gpu]
export HF_TOKEN="your-huggingface-token" # get yours at https://huggingface.co/settings/tokens
helm install ${PROJECT} helm/ --namespace ${PROJECT} --set device=${DEVICE} --set sa=${SA} --set hfToken=${HF_TOKEN}
```



## Test

TODO: explain how to access the UI, give sample simple and complex queries of different types, show which hardware is selected, and include screenshots of sample outputs

## Delete

To uninstall and delete the project:

```bash
helm uninstall confidential-ai
oc delete project confidential-ai-demo
```

## References 

TODO: optional
