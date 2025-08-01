# 🎯 GPU-as-a-Service on OpenShift AI — Full Demo Runbook

A complete end-to-end implementation of GPUaaS using Red Hat OpenShift AI (RHODS), NVIDIA V100 GPUs, MinIO, KServe with vLLM runtime, Elyra pipelines, and GPU monitoring via Prometheus & Grafana.

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

- Install `Node Feature Discovery Operator` from OperatorHub
- Apply node label for GPU scheduling:

```bash
oc label node <gpu-node-name> feature.node.kubernetes.io/custom-gpu=true
```

✅ **Verification**:
```bash
oc get nodes --show-labels | grep custom-gpu
```

---

### 🔌 Step 2: Install NVIDIA GPU Operator

- Install `NVIDIA GPU Operator` via OperatorHub (namespace: `nvidia-gpu-operator`)
- Ensure DaemonSet pods are running:

✅ **Verification**:
```bash
oc get pods -n nvidia-gpu-operator
nvidia-dcgm-exporter-xxxxx     Running
nvidia-driver-daemonset-xxxxx  Running
```

---

### ☁️ Step 3: Install MinIO (Open Source) & Create Bucket

- Deploy using Helm or Operator (we used Helm)
- Access UI: `https://minio-minio.apps.<cluster-domain>`
- Create bucket: `onnx-models`

✅ **Verification**:
- UI bucket visible
- Upload test `.onnx` model
- S3 endpoint:
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

### 🧠 Step 5: Launch GPU-backed Workbench in RHODS

- Project: `gpu-dsp`
- Image: `cuda-py39`
- Resources: 1 GPU, 8Gi RAM
- Attached Elyra extension

✅ **Notebook Ready**:
- Elyra visible
- GPU available via `nvidia-smi`

---

### 📦 Step 6: GPU Monitoring via DCGM + Prometheus + Grafana

- Deploy `dcgm-exporter` as DaemonSet
- Add `ServiceMonitor` + Grafana datasource
- Import `nvidia-dcgm-gpu-dashboard.json`

✅ **Sample Metrics**:
- `DCGM_FI_DEV_GPU_UTIL`
- `DCGM_FI_DEV_MEM_COPY_UTIL`

---

### 🧪 Step 7: MLOps Pipeline with Elyra

Directory: `pipelines/elyra/`

- `01_train_model.ipynb`: Simulated ONNX model training
- `02_upload_to_minio.ipynb`: Upload to `onnx-models` via boto3
- `03_deploy_kserve.ipynb`: Apply `inference-service-vllm.yaml`
- `04_test_inference.ipynb`: Run POST request on KServe URL

✅ **Dry Run Outputs**:
- ✅ Model trained or loaded
- ✅ Model uploaded to MinIO
- ✅ InferenceService applied via `oc`
- ✅ HTTP inference returns:
  ```json
  {"predictions": [0.812]}
  ```

---

### 📡 Step 8: vLLM + KServe Deployment

- Runtime: `kserve-onnxruntime`
- ModelFormat: `onnx`
- Access via:
  ```
  https://onnx-model-vllm-gpu-dsp.apps.<cluster-domain>
  ```

✅ **Verify Deployment**:
```bash
oc get inferenceservice onnx-model-vllm -n gpu-dsp
```

---

## 📂 Directory Structure

```
gpuaaas-openshift-demo/
├── pipelines/elyra/         # Elyra notebooks + pipeline
├── monitoring/              # Dashboards and exporters
├── yamls/                   # InferenceService and configs
└── README.md
```

---

## 🧪 Validation Checklist

| Item                          | Status     |
|-------------------------------|------------|
| OpenShift AI Installed        | ✅ Complete |
| GPU Operator Running          | ✅ Complete |
| MinIO Bucket Ready            | ✅ Complete |
| GPU Workbench Functional      | ✅ Complete |
| Elyra Pipeline Functional     | ✅ Complete |
| InferenceService Available    | ✅ Complete |
| GPU Dashboard in Grafana      | ✅ Complete |

---

## 📝 Next Steps

- Add Tekton pipeline CI/CD
- Integrate Seldon or BentoML
- Enable autoscaling with Cluster Autoscaler
- Monitor cost with Cost Operator and GPU labels

---

© 2025 Red Hat | msugur@redhat.com