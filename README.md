
# GPU-as-a-Service (GPUaaS) on Red Hat OpenShift AI - Beginner-Friendly Implementation Runbook

## Overview
This runbook is designed for users **new to Kubernetes and GPU infrastructure**. It walks through deploying a complete GPUaaS demo on OpenShift AI with NVIDIA GPUs, model inferencing using KServe, object storage with MinIO, and GPU monitoring using Prometheus and Grafana.

---

## Prerequisites
- OpenShift 4.15+ cluster with admin access
- 2+ worker nodes with NVIDIA V100 GPUs
- NVIDIA GPU Operator entitlement
- RHODS (Red Hat OpenShift AI) installed
- `oc` CLI tool configured
- Basic Linux terminal knowledge

---

## STEP 1: Validate GPU Nodes

**1.1 Check if nodes have GPUs:**
```bash
oc get nodes -l nvidia.com/gpu.present=true
```

**1.2 Inspect node GPU labels:**
```bash
oc describe node <gpu-node-name> | grep -i nvidia
```

---

## STEP 2: Install Node Feature Discovery (NFD)

**2.1 Install via OperatorHub:**
- Navigate to Operators > OperatorHub
- Search for **Node Feature Discovery**
- Install in the default namespace

**2.2 Label GPU nodes:**
```bash
oc label node <gpu-node-name> feature.node.kubernetes.io/custom-gpu=true
```

---

## STEP 3: Install NVIDIA GPU Operator

**3.1 Install via OperatorHub:**
- Search for **NVIDIA GPU Operator**
- Create a new project: `nvidia-gpu-operator`
- Approve installation

**3.2 Validate:**
```bash
oc get pods -n nvidia-gpu-operator
```
You should see pods like `nvidia-driver-daemonset` and `nvidia-dcgm-exporter`.

---

## STEP 4: Install MinIO (Object Storage)

**4.1 Deploy MinIO open source manually:**
```bash
kubectl create namespace minio
kubectl apply -f https://raw.githubusercontent.com/minio/minio/master/docs/orchestration/kubernetes/minio-standalone.yaml -n minio
```

**4.2 Access MinIO Console:**
- Route: `https://minio-minio.apps.<your-cluster-domain>`
- Login with: `minioadmin:minioadmin`

**4.3 Create a bucket:**
- Name: `onnx-models`
- Upload your model: `model.onnx`

---

## STEP 5: Create S3 Secret for MinIO

```bash
oc create secret generic s3storageconfig   --from-literal=AWS_ACCESS_KEY_ID=minioadmin   --from-literal=AWS_SECRET_ACCESS_KEY=minioadmin -n gpu-dsp
```

---

## STEP 6: Launch GPU Workbench

**6.1 Create a new project:**
```bash
oc new-project gpu-dsp
```

**6.2 Launch workbench via RHODS:**
- Image: `cuda-py39` or similar
- Resources: 1 GPU, 8GiB RAM, 2 CPUs
- Enable GPU accelerator

**6.3 Verify inside terminal:**
```bash
!nvidia-smi
```

---

## STEP 7: Deploy Inference Service with KServe

**7.1 Update YAML (inference-service-vllm.yaml):**
```yaml
spec:
  predictor:
    model:
      modelFormat:
        name: onnx
      storage:
        key: s3storageconfig
        path: s3://onnx-models/model.onnx
```

**7.2 Apply YAML:**
```bash
oc apply -f yamls/inference-service-vllm.yaml -n gpu-dsp
```

**7.3 Validate:**
```bash
oc get inferenceservice -n gpu-dsp
```

---

## STEP 8: Send Inference Request

**8.1 Use curl or Python:**
```bash
curl -X POST   http://onnx-vllm-inference-gpu-dsp.apps.<cluster>/v1/models/onnx-vllm-inference:predict   -H "Content-Type: application/json"   -d @payloads/sample-inference-payload.json
```

---

## STEP 9: GPU Monitoring with Prometheus + Grafana

**9.1 Apply monitoring stack:**
```bash
oc apply -f monitoring/gpu-monitoring-bundle.yaml -n gpu-dsp
```

**9.2 Access Grafana Dashboard:**
- Route: OpenShift > Monitoring > Dashboards
- Dashboard: NVIDIA GPU Monitoring

---

## STEP 10: Validate All Components

Use `notebooks/final-validation-checks.ipynb` in your workbench to:
- Verify GPU usage
- Confirm model endpoints
- Test inference
- View GPU metrics

---

## ✅ Success Checklist
| Component             | Status     |
|----------------------|------------|
| GPU Nodes Recognized | ✅         |
| NFD Installed        | ✅         |
| GPU Operator Active  | ✅         |
| MinIO Configured     | ✅         |
| Inference Running    | ✅         |
| Grafana Dashboards   | ✅         |

---

## 📝 Summary
You have successfully deployed a GPU-as-a-Service architecture using:
- OpenShift AI (RHODS)
- NVIDIA GPU Operator
- KServe with vLLM
- Object storage (MinIO)
- Monitoring (DCGM + Prometheus + Grafana)

This setup supports further enhancements like Elyra pipelines, Tekton CI/CD, and MIG for NVIDIA A100s.

---

For support, contact: msugur@redhat.com
