# MLOps — Vận hành hệ thống ML/AI (hướng gần bạn nhất)

> MLOps = áp dụng tư duy DevOps cho **vòng đời Machine Learning**. Nhưng ML khác phần mềm truyền thống
> ở chỗ: sản phẩm không chỉ là **code** mà là **code + data + model** — cả ba đều thay đổi và đều phải versioned,
> tested, deployed, monitored. Đây là lý do MLOps là một chuyên môn riêng, không chỉ "DevOps cho AI".

## 1. Vì sao ML khó hơn phần mềm thường (mindset)

```text
Phần mềm thường:  code -> build -> test -> deploy -> chạy ổn định (behavior xác định)
ML system:        code + DATA + MODEL -> train -> eval -> deploy -> DRIFT theo thời gian

3 nguồn thay đổi (thay vì 1):
  - Code thay đổi     (như DevOps thường)
  - DATA thay đổi     (phân phối dữ liệu thực tế trôi đi -> data drift)
  - MODEL thay đổi    (retrain, đổi hyperparameter, đổi kiến trúc)

Hệ quả: "CI/CD" của ML phức tạp hơn nhiều -> CI/CD/CT (Continuous Training).
```
**Technical debt đặc thù ML** (paper Google "Hidden Technical Debt in ML Systems"): code ML chỉ là phần nhỏ; phần lớn là data pipeline, config, monitoring, serving infra. Đây chính là **đất diễn của MLOps** — và là nơi nền DevOps của bạn có giá trị trực tiếp.

## 2. Vòng đời ML & các thành phần MLOps

```text
[Data] -> [Feature Eng] -> [Train] -> [Evaluate] -> [Register] -> [Deploy/Serve] -> [Monitor] -> (loop lại)
   |            |             |           |             |              |               |
 versioning  feature      experiment   metrics/     model         serving         drift/perf
 (DVC/       store        tracking     validation   registry      (online/batch)  monitoring
  lakeFS)    (Feast)     (MLflow/W&B)                (MLflow)                       -> trigger retrain
```

### Các trụ cột (mỗi cái là 1 mảng để nắm)
| Trụ cột | Vấn đề giải quyết | Công cụ tiêu biểu |
|---------|-------------------|-------------------|
| **Data versioning** | version data như version code | DVC, lakeFS, Delta Lake, Git LFS |
| **Feature Store** | tái dùng feature, đồng nhất train/serve | Feast, Tecton, SageMaker FS |
| **Experiment tracking** | log param/metric/artifact mỗi lần train | MLflow, Weights & Biases, Neptune |
| **Pipeline orchestration** | tự động hóa train pipeline | Kubeflow Pipelines, Airflow, Metaflow, Dagster, Argo Workflows |
| **Model registry** | quản lý version model + stage (staging/prod) | MLflow Registry, SageMaker |
| **Model serving** | expose model qua API (online) / batch | KServe, Seldon, BentoML, TorchServe, Triton, SageMaker Endpoints |
| **Monitoring** | drift, data quality, model perf, latency | Evidently, WhyLabs, Arize, Prometheus/Grafana |
| **CI/CD/CT** | test + deploy + retrain tự động | GitHub Actions + các cái trên, Kubeflow |

## 3. Kỹ năng: bạn ĐÃ CÓ vs CẦN THÊM

**Bạn đã có (nền DevOps — chuyển thẳng sang được):**
- Kubernetes, container, cloud, IaC, CI/CD, observability → **80% hạ tầng MLOps chạy trên chính những thứ này.**
- Serving model = deploy container có health check/autoscale (bạn làm được).
- Pipeline = orchestration (giống CI/CD bạn quen).

**Cần bổ sung (phần "ML-specific"):**
```text
1. Hiểu vòng đời ML đủ để nói chuyện với Data Scientist (không cần train model giỏi,
   nhưng phải hiểu: train/eval/inference, overfitting, metric, feature, dataset split).
2. Data engineering cơ bản: pipeline dữ liệu, data quality, versioning, feature store.
3. Model serving patterns: online vs batch, real-time vs async, GPU serving, batching,
   model optimization (quantization, ONNX, TensorRT) để giảm latency/cost.
4. ML monitoring: data drift, concept drift, training-serving skew, model performance decay.
5. Experiment tracking & reproducibility: seed, môi trường, data version -> tái tạo được kết quả.
6. GPU infrastructure: scheduling GPU trên K8s (nvidia device plugin), chia sẻ GPU (MIG,
   time-slicing), tối ưu chi phí GPU (đắt nhất trong ML infra).
```

## 4. LLMOps — nhánh nóng nhất (rất liên quan nếu bạn làm AI hiện đại)

Nếu dự án AI của bạn dùng LLM (GPT/Claude/Llama...), có một nhánh riêng **LLMOps** với thách thức khác classic ML:

```text
Classic ML: bạn train model từ data của mình -> lo train pipeline.
LLMOps:     thường DÙNG model có sẵn (API hoặc open-weight) -> lo:
```
| Chủ đề LLMOps | Nội dung |
|---------------|----------|
| **Prompt engineering & versioning** | prompt là "code" mới — cần version, test, eval |
| **RAG (Retrieval-Augmented Generation)** | vector DB (pgvector, Pinecone, Weaviate, Milvus), embedding pipeline, chunking |
| **Fine-tuning / PEFT (LoRA)** | tinh chỉnh model mở khi cần; quản lý adapter |
| **Serving LLM** | vLLM, TGI, Ollama; tối ưu KV-cache, batching, throughput; GPU memory |
| **Evaluation** | LLM khó eval (không có 1 đáp án đúng) → LLM-as-judge, eval set, human feedback |
| **Guardrails** | chống prompt injection, lọc output độc hại, PII |
| **Cost & latency** | token cost, caching, routing model (rẻ vs mạnh), streaming |
| **Observability** | trace prompt→response, log token, đánh giá chất lượng online (LangSmith, Langfuse, Phoenix) |

> Đây là nơi **cực kỳ thiếu người** biết cả hạ tầng lẫn LLM. Nếu bạn đã làm AI, đào sâu LLMOps là đòn bẩy lớn nhất.

## 5. Kiến trúc MLOps mẫu (trên nền bạn đã biết)

```text
Data:      S3/lakeFS (versioned) -> Feature Store (Feast)
Train:     Kubeflow/Argo Workflows trên K8s (GPU node pool) -> log vào MLflow
Registry:  MLflow Model Registry (staging -> prod, có approval)
CI/CD/CT:  Git push -> test data+model -> train -> eval gate (metric đạt ngưỡng?) ->
           register -> deploy KServe (canary) -> monitor -> drift trigger retrain
Serving:   KServe/BentoLM trên K8s, autoscale (kể cả scale-to-zero), GPU, Triton/vLLM
Monitor:   Prometheus/Grafana (latency, GPU util) + Evidently (drift, data quality)
           -> alert -> retrain pipeline
GitOps:    ArgoCD quản lý toàn bộ manifest (bạn đã biết)
```
→ Nhìn kỹ: **hạ tầng bên dưới là K8s + GitOps + CI/CD + observability = đúng thứ bạn đã có.** Phần thêm là các lớp ML-specific ở trên.

## 6. Lộ trình học (tận dụng nền DevOps)

```text
Giai đoạn 1 — Ngôn ngữ chung với Data Scientist (2-4 tuần)
  - Hiểu vòng đời ML, train/eval/inference, metric cơ bản, overfitting.
  - Chạy 1 model đơn giản end-to-end (sklearn/pytorch) để hiểu DS làm gì.
  - Học MLflow: track experiment + registry.

Giai đoạn 2 — Hạ tầng MLOps (nền bạn mạnh) (4-6 tuần)
  - Đóng gói model thành service (BentoML/FastAPI) -> deploy K8s (bạn làm được).
  - Model serving: KServe hoặc Seldon; autoscale; GPU scheduling trên K8s.
  - Pipeline: Kubeflow Pipelines hoặc Argo Workflows cho train/retrain.
  - Data/feature: DVC + Feast.

Giai đoạn 3 — Vận hành ML (SRE cho ML) (4 tuần)
  - Monitoring: Evidently (drift) + Prometheus (infra) + alert -> retrain.
  - CI/CD/CT: gate theo metric, canary deploy model, rollback.
  - Tối ưu GPU cost.

Giai đoạn 4 — LLMOps (nếu làm LLM) (4-6 tuần)
  - RAG end-to-end: embedding -> vector DB -> retrieval -> LLM.
  - Serving LLM open-weight (vLLM), tối ưu throughput/latency.
  - Eval + guardrails + observability (Langfuse/Phoenix) + cost control.

Capstone: dựng 1 ML platform hoàn chỉnh (data -> train -> serve -> monitor -> retrain)
          bằng IaC + GitOps. Đây là hồ sơ vàng.
```

## 7. Thị trường & đòn bẩy sự nghiệp
- **Cầu >> cung.** Rất ít người biết cả DevOps/hạ tầng lẫn ML. Bạn đã có nửa hiếm (hạ tầng) + đang làm AI → lợi thế lớn.
- Chức danh: MLOps Engineer, ML Platform Engineer, **ML Infrastructure Engineer**, AI Platform Engineer, LLMOps Engineer.
- Lương thường cao hơn DevOps thuần vì độ hiếm + tác động trực tiếp tới sản phẩm AI (nguồn doanh thu).
- **Đây là con đường IC/Architect tự nhiên nhất cho bạn**: ML Platform Architect.

## 8. Giao thoa với SRE & DevSecOps
- **↔ SRE:** serving ML = bài toán reliability (SLO cho latency/accuracy, capacity GPU, incident khi model degrade). Xem [sre.md](./sre.md).
- **↔ DevSecOps:** **MLSecOps** — bảo mật data/model, model supply chain (model từ đâu, có bị đầu độc không), LLM security (prompt injection), governance & compliance (GDPR cho training data, model lineage/audit). Xem [devsecops.md](./devsecops.md).

## Tóm tắt
- MLOps quản **code + data + model** (3 nguồn thay đổi), thêm **Continuous Training** vào CI/CD.
- Nền DevOps của bạn lo ~80% hạ tầng; phần thêm là data/model/serving/drift.
- Nếu làm LLM → **LLMOps** là đòn bẩy nóng nhất (RAG, serving, eval, guardrails, cost).
- Hướng phù hợp nhất với bạn: **ML/AI Platform Engineer** — hiếm, giá trị cao, đúng làn sóng.
