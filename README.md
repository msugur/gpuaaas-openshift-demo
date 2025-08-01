# 🎯 GPU-as-a-Service on OpenShift AI — Full Demo Runbook (Without Elyra)

A complete GPUaaS demo on Red Hat OpenShift AI (RHODS) using NVIDIA V100 GPUs, MinIO, KServe with vLLM runtime, and GPU monitoring via Prometheus & Grafana.

---

## 🧭 Prerequisites

- OpenShift 4.15+
- 2+ Worker nodes with NVIDIA V100 GPUs
- RHODS Operator installed
- `oc` CLI configured
- Access to GPU-backed Workbench

---

## 🔢 Step-by-Step Execution with Verification & Outputs

---

### 🔧 Step 1: Install Node Feature Discovery (NFD)

Install **Node Feature Discovery Operator** from OperatorHub.

Label your GPU node:

```bash
oc label node <gpu-node-name> feature.node.kubernetes.io/custom-gpu=true
```

✅ **Verification**:
```bash
oc get nodes --show-labels | grep custom-gpu
```

---

### 🔌 Step 2: Install NVIDIA GPU Operator

Install **GPU Operator** in `nvidia-gpu-operator` namespace.

✅ **Verification**:
```bash
oc get pods -n nvidia-gpu-operator
nvidia-driver-daemonset-*      Running
nvidia-dcgm-exporter-*         Running
```

---

### ☁️ Step 3: Install MinIO (Open Source) & Create Bucket

- Deploy MinIO via Helm or static manifests.
- Console UI: `https://minio-minio.apps.<cluster-domain>`
- Create a bucket: `onnx-models`
- Upload your model file (e.g., `model.onnx`)

✅ **Verify**:
- Accessible at:
  ```
  https://minio-api-minio.apps.<cluster-domain>
  ```

---

### 🔐 Step 4: Create MinIO S3 Secret

```bash
oc create secret generic s3storageconfig \
  --from-literal=AWS_ACCESS_KEY_ID=minioadmin \
  --from-literal=AWS_SECRET_ACCESS_KEY=minioadmin
```

✅ **Verify Secret**:
```bash
oc get secret s3storageconfig -o yaml
```

---

### 🧠 Step 5: Launch GPU Workbench (RHODS)

- Namespace: `gpu-dsp`
- Environment: `cuda-py39` image
- Allocate: 1 GPU, 8GiB RAM

✅ Inside Jupyter:
```python
!nvidia-smi
```

---

### 📈 Step 6: GPU Monitoring with Prometheus & Grafana

1. Deploy `dcgm-exporter` as a DaemonSet
2. Add `ServiceMonitor` and Grafana config
3. Import dashboard JSON for DCGM metrics

✅ Key Metrics:
- `DCGM_FI_DEV_GPU_UTIL`
- `DCGM_FI_DEV_MEM_COPY_UTIL`

---

### 🚀 Step 7: Deploy InferenceService using vLLM + KServe

Use `inference-service-vllm.yaml` to deploy the ONNX model:

```bash
oc apply -f inference-service-vllm.yaml -n gpu-dsp
```

✅ **Expected Output**:
```bash
NAME                URL                                                      READY
onnx-model-vllm     https://onnx-model-vllm-gpu-dsp.apps.<domain>           True
```

Test inference with:

```bash
curl -X POST <inference-url> -H "Content-Type: application/json" -d '{"inputs": [1.0, 2.0]}'
```

---

## 📂 Directory Layout

```
gpuaaas-openshift-demo/
├── yamls/
│   └── inference-service-vllm.yaml
├── monitoring/
│   ├── grafana-dashboards/
│   └── dcgm-exporter/
├── README.md
```

---

## ✅ Validation Checklist

| Component                  | Status     |
|---------------------------|------------|
| NFD Installed             | ✅ Complete |
| GPU Operator Running      | ✅ Complete |
| MinIO Configured          | ✅ Complete |
| S3 Secret Configured      | ✅ Complete |
| GPU Workbench Launched    | ✅ Complete |
| KServe Deployed           | ✅ Complete |
| Inference Working         | ✅ Complete |
| Monitoring Enabled        | ✅ Complete |

---

## 📝 Next Enhancements

- Enable Tekton CI/CD Pipelines
- Add MIG setup for A100 GPUs
- Automate via ArgoCD or GitOps
- Extend to multi-tenant GPUaaS

---

© 2025 Red Hat | msugur@redhat.com