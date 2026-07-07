# ML Serving trên Kubernetes (KServe + GPU) — thế mạnh khác biệt của bạn

> Đây là nơi nền DevOps/K8s của bạn tạo giá trị mà Data Scientist thường không làm được:
> đưa model lên production **đáng tin, tự co giãn, tối ưu GPU**. File này đi sâu kỹ thuật với manifest thật.

## 📑 Nội dung
- [1. Bức tranh serving](#1-bức-tranh-serving)
- [2. Online vs Batch vs Streaming](#2-online-vs-batch-vs-streaming)
- [3. KServe — kiến trúc & InferenceService](#3-kserve--kiến-trúc--inferenceservice)
- [4. GPU trên Kubernetes](#4-gpu-trên-kubernetes)
- [5. Tối ưu inference (latency & cost)](#5-tối-ưu-inference-latency--cost)
- [6. Autoscaling (kể cả scale-to-zero)](#6-autoscaling-kể-cả-scale-to-zero)
- [7. Rollout an toàn (canary) & rollback](#7-rollout-an-toàn-canary--rollback)
- [8. Observability cho serving](#8-observability-cho-serving)
- [9. Checklist production](#9-checklist-production-cho-model-serving)

---

## 1. Bức tranh serving

```text
Model đã train (registry/S3) -> đóng gói -> serving runtime -> expose API -> client
                                              |
                        + preprocess/postprocess (transformer)
                        + autoscale, GPU, batching, canary, monitoring
```
Hai cách chính:
- **Tự đóng gói** (FastAPI/BentoML + container): kiểm soát tối đa, hợp logic phức tạp.
- **Serving platform** (KServe/Seldon): chuẩn hoá, có sẵn autoscale/canary/monitoring/GPU — **khuyên dùng cho production**.

## 2. Online vs Batch vs Streaming

| Kiểu | Đặc điểm | Dùng khi | Hạ tầng |
|------|----------|----------|---------|
| **Online (real-time)** | request → response ngay, latency thấp | API, gợi ý realtime, fraud check | KServe/Seldon, autoscale, GPU nếu cần |
| **Batch** | chạy định kỳ trên khối dữ liệu lớn | scoring hàng loạt, báo cáo | Job/CronJob, Argo Workflows, Spark |
| **Streaming** | xử lý theo luồng sự kiện | IoT, realtime pipeline | Kafka + consumer, Flink |

**Điểm quyết định:** latency yêu cầu + tần suất. Cần trả lời trong ms cho từng request → online. Chấm điểm 10 triệu bản ghi mỗi đêm → batch (rẻ hơn nhiều, không cần serve 24/7).

## 3. KServe — kiến trúc & InferenceService

KServe (trên K8s + Knative) chuẩn hoá serving: 1 CRD `InferenceService` lo autoscale, canary, GPU, logging.

```text
InferenceService
  ├── Predictor      (bắt buộc) — chạy model (sklearn/pytorch/triton/vLLM/custom)
  ├── Transformer    (tùy chọn) — pre/post-process (tokenize, chuẩn hoá ảnh...)
  └── Explainer      (tùy chọn) — giải thích dự đoán
Traffic -> (Transformer ->) Predictor. Mỗi phần tự autoscale riêng.
```

### Ví dụ InferenceService cơ bản (model từ S3)
```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: sklearn-iris
  namespace: ml-prod
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 5
    scaleTarget: 50          # target concurrency mỗi pod
    model:
      modelFormat: { name: sklearn }
      storageUri: s3://my-models/iris/v3/    # trỏ tới model đã register
      resources:
        requests: { cpu: "1", memory: 2Gi }
        limits:   { cpu: "2", memory: 4Gi }
```
```bash
kubectl get inferenceservice -n ml-prod          # READY=True là xong, có URL
# Gọi thử:
curl -X POST http://<url>/v1/models/sklearn-iris:predict \
  -d '{"instances": [[5.1, 3.5, 1.4, 0.2]]}'
```

### Ví dụ có GPU (PyTorch/Triton)
```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata: { name: resnet-gpu, namespace: ml-prod }
spec:
  predictor:
    minReplicas: 0                 # scale-to-zero khi rảnh (tiết kiệm GPU)
    maxReplicas: 4
    model:
      modelFormat: { name: pytorch }
      storageUri: s3://my-models/resnet/v2/
      resources:
        requests: { cpu: "2", memory: 8Gi, nvidia.com/gpu: "1" }
        limits:   { cpu: "4", memory: 16Gi, nvidia.com/gpu: "1" }
    nodeSelector: { "nvidia.com/gpu.product": "A10G" }   # chọn loại GPU
    tolerations:
      - { key: "nvidia.com/gpu", operator: "Exists", effect: "NoSchedule" }
```

### Custom Predictor (khi cần logic riêng)
Đóng model + code bằng container của bạn (FastAPI/BentoML), KServe vẫn lo autoscale/canary:
```yaml
spec:
  predictor:
    containers:
      - name: kserve-container
        image: registry.example.com/my-model-server:1.4.0
        ports: [{ containerPort: 8080 }]
        resources:
          requests: { nvidia.com/gpu: "1", memory: 8Gi }
```

## 4. GPU trên Kubernetes

### Điều kiện để pod dùng được GPU
```text
1. Node có GPU + NVIDIA driver.
2. Cài NVIDIA device plugin (hoặc GPU Operator - khuyên dùng, tự lo driver+plugin+monitoring):
      kubectl create -f https://.../nvidia-gpu-operator ...
3. Pod request tài nguyên: resources.limits."nvidia.com/gpu": "1"
   (GPU là integer, KHÔNG chia lẻ như CPU trừ khi bật MIG/time-slicing)
4. Đặt GPU node vào node pool riêng + taint để pod thường không chiếm.
```
```bash
kubectl get nodes -o custom-columns='NAME:.metadata.name,GPU:.status.capacity.nvidia\.com/gpu'
kubectl describe node <gpu-node> | grep nvidia.com/gpu   # allocatable vs allocated
```

### Chia sẻ GPU (GPU đắt → tối ưu utilization)
| Cách | Cơ chế | Dùng khi |
|------|--------|----------|
| **Time-slicing** | nhiều pod chia sẻ 1 GPU theo thời gian | nhiều model nhỏ, tải thấp, không cần cách ly |
| **MIG (Multi-Instance GPU)** | chia 1 GPU (A100/H100) thành nhiều instance cách ly phần cứng | cần cách ly + đảm bảo tài nguyên |
| **MPS** | chạy song song nhiều process trên 1 GPU | throughput cao, cùng tin cậy |

> GPU là chi phí lớn nhất của ML infra. Biết **chia sẻ GPU + scale-to-zero** là kỹ năng tiết kiệm tiền cực giá trị (điểm bạn ghi điểm với sếp).

### Chống lãng phí GPU
```text
- scale-to-zero (minReplicas: 0) cho model tải thấp/không đều.
- Node pool GPU riêng + Cluster Autoscaler/Karpenter (thêm/bớt GPU node theo tải).
- Spot GPU instance cho batch/inference chịu gián đoạn (rẻ hơn nhiều).
- Right-size: model nhỏ dùng GPU nhỏ (T4/A10G) thay vì A100.
- Giám sát GPU util (DCGM exporter) -> GPU <30% util kéo dài = đang đốt tiền.
```

## 5. Tối ưu inference (latency & cost)

```text
1. Model optimization:
   - Quantization (FP32 -> FP16/INT8): nhỏ hơn, nhanh hơn, ít mất chất lượng.
   - ONNX Runtime / TensorRT: biên dịch tối ưu cho phần cứng.
   - Distillation: model nhỏ học từ model lớn.
2. Dynamic batching: gom nhiều request thành 1 batch -> tận dụng GPU (Triton/vLLM làm sẵn).
3. Model server hiệu năng cao: Triton Inference Server (đa framework, batching, multi-model).
4. Caching: cache kết quả cho input trùng.
5. Chọn phần cứng đúng: CPU đủ cho model nhỏ; GPU cho model lớn/throughput cao.
```
**Ví dụ tác động:** một model chuyển FP16 + dynamic batching + TensorRT có thể giảm latency 3-5x và tăng throughput nhiều lần trên cùng GPU → giảm số GPU cần → giảm tiền.

## 6. Autoscaling (kể cả scale-to-zero)

```text
KServe/Knative scale theo CONCURRENCY hoặc RPS (không chỉ CPU) -> hợp với inference.
  scaleTarget: 50   -> mỗi pod xử lý ~50 request đồng thời, vượt thì scale thêm.
  minReplicas: 0    -> scale-to-zero khi không có traffic (tiết kiệm, nhưng có cold start).

Với model không thể scale-to-zero (cold start lâu): minReplicas >= 1.
Với custom metric (queue length, GPU util) -> KEDA.
```
- **Cold start** của model lớn (nạp GB weight vào GPU) có thể lâu → cân nhắc giữ tối thiểu 1 replica cho model quan trọng, hoặc pre-warm.

## 7. Rollout an toàn (canary) & rollback

KServe hỗ trợ canary bằng `canaryTrafficPercent` — dịch traffic dần sang version mới:
```yaml
spec:
  predictor:
    canaryTrafficPercent: 10          # 10% traffic sang model mới, 90% giữ model cũ
    model:
      storageUri: s3://my-models/iris/v4/   # version mới
```
```text
Quy trình an toàn:
  10% -> theo dõi (latency, error, chất lượng) -> 50% -> 100%.
  Nếu metric xấu -> đặt canaryTrafficPercent về 0 (rollback tức thì về model cũ).
```
> Đây chính là tư duy Blue/Green-Canary bạn đã quen từ DevOps, áp cho model.

## 8. Observability cho serving

```text
Infra (bạn đã có):
  - Prometheus: request rate, latency (p50/p95/p99), error rate, replica count.
  - GPU: DCGM exporter -> GPU util, memory, temperature, power.
  - KServe expose sẵn metrics -> scrape vào Prometheus -> Grafana dashboard.
ML-specific (thêm):
  - Prediction logging: log input/output (lấy mẫu) -> phân tích drift sau.
  - Data drift: Evidently so phân phối input serving vs training.
  - Model quality: nếu có ground-truth trễ -> đo accuracy online.
Alert: latency p99 cao, error tăng, GPU OOM, drift vượt ngưỡng, GPU util thấp kéo dài (lãng phí).
```

## 9. Checklist production cho model serving
```text
[ ] Model từ registry có version rõ (không "latest").
[ ] resources.requests/limits đặt đúng (kể cả nvidia.com/gpu).
[ ] Autoscale cấu hình (min/max, scaleTarget); cân nhắc scale-to-zero.
[ ] GPU node pool riêng + taint/toleration; chia sẻ GPU nếu hợp.
[ ] Tối ưu inference (quantize/ONNX/batching) để giảm latency & GPU cần.
[ ] Canary + rollback tức thì (canaryTrafficPercent).
[ ] Readiness/liveness probe (model nạp xong mới nhận traffic).
[ ] Observability: latency/error/GPU util + prediction logging + drift.
[ ] Alert đầy đủ (gồm cả "GPU lãng phí").
[ ] Manifest trong Git (GitOps/ArgoCD), không sửa tay.
[ ] Load test trước khi lên (biết ngưỡng chịu tải).
```

---

### Vì sao đây là thế mạnh của bạn
Data Scientist train model giỏi nhưng thường **không** biết: GPU scheduling, autoscale theo concurrency, canary, tối ưu inference, cost GPU, observability production. Đây đúng là nền DevOps của bạn áp vào ML → bạn thành **cầu nối không thể thiếu** giữa DS và production. Nắm chắc file này là bạn đã có năng lực lõi của một ML Platform Engineer.
