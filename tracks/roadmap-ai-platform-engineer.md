# Lộ trình: DevOps → AI Platform Engineer (MLOps + SRE + DevSecOps)

> Lộ trình phân lớp cho người đang làm **DevOps cho dự án AI**: lấy **MLOps làm lõi**, bồi thêm
> **SRE** (độ tin cậy) và **DevSecOps** (bảo mật), để thành **AI/ML Platform Engineer** — hồ sơ IC/Architect hiếm.
> Đọc kèm: [mlops.md](./mlops.md) · [sre.md](./sre.md) · [devsecops.md](./devsecops.md)

## Phần 1 — LLMOps khác gì MLOps?

**MLOps** = vận hành TOÀN BỘ ML (bao trùm). **LLMOps** = một **nhánh con chuyên biệt** của MLOps cho các mô hình ngôn ngữ lớn. Khác biệt đến từ chỗ: MLOps cổ điển thường **bạn tự train model từ data của mình**, còn LLMOps thường **dùng lại model khổng lồ có sẵn** (API hoặc open-weight) và tùy biến quanh nó.

```text
MLOps cổ điển:  DATA của bạn -> TRAIN model -> deploy -> monitor -> retrain
                (trọng tâm: train pipeline, feature, dataset)

LLMOps:         DÙNG model nền có sẵn (GPT/Claude/Llama) -> tùy biến bằng prompt/RAG/fine-tune
                -> deploy/gọi API -> eval chất lượng -> guardrails -> tối ưu cost/latency
                (trọng tâm: prompt, context, retrieval, eval, cost token)
```

### Bảng so sánh cốt lõi

| Khía cạnh | **MLOps (cổ điển)** | **LLMOps** |
|-----------|---------------------|------------|
| Nguồn model | Bạn train từ đầu | Model nền có sẵn (foundation model) |
| "Code" chính | Training code, feature | **Prompt, chain, retrieval config** |
| Dữ liệu | Training dataset (labeled) | Context/knowledge base (RAG), ít/không cần label |
| Tùy biến | Train/retrain | Prompt engineering → RAG → fine-tune (PEFT/LoRA) |
| Compute nặng nhất | **Training** (GPU cho train) | **Inference/serving** (GPU cho serve, model rất lớn) |
| Đánh giá (eval) | Metric rõ (accuracy, F1, AUC) | **Khó** — không 1 đáp án đúng → LLM-as-judge, human eval, eval set |
| Chi phí chính | GPU train + storage | **Token cost** (API) hoặc GPU serve (self-host) |
| Rủi ro đặc thù | Data/concept drift | **Hallucination, prompt injection, jailbreak, data leakage** |
| Monitoring | Drift, model perf | Chất lượng output, token, cost, latency, an toàn nội dung |
| Versioning | data + model | **prompt + model + retrieval index + config** |
| Công cụ đặc thù | MLflow, Kubeflow, Feast | Vector DB, LangChain/LlamaIndex, vLLM/TGI, Langfuse/LangSmith |

### Điểm mấu chốt để định hình
- LLMOps **KHÔNG thay thế** MLOps — nó **ngồi trên nền MLOps**. Bạn vẫn cần hạ tầng serving, CI/CD, monitoring, versioning của MLOps; LLMOps thêm các lớp: prompt/RAG/eval/guardrails/token-cost.
- Nếu dự án AI của bạn **dùng LLM (API hoặc open-weight)** → LLMOps là phần bạn chạm hằng ngày, học nó trước cũng được.
- Nếu dự án **train model riêng (vision, tabular, recommendation...)** → MLOps cổ điển là trọng tâm.
- **Chiến lược an toàn:** nắm **nền MLOps chung** trước (versioning, serving, pipeline, monitoring) rồi rẽ nhánh LLMOps ở lớp trên. Nền chung dùng được cho cả hai.

---

## Phần 2 — Chiến lược phân lớp (đọc trước khi vào lộ trình)

```text
        ┌─────────────────────────────────────────────┐
        │  LỚP 3: DevSecOps / MLSecOps (bảo mật)        │  <- bồi sau cùng
        ├─────────────────────────────────────────────┤
        │  LỚP 2: SRE cho ML (độ tin cậy, SLO, GPU)    │  <- bồi thứ hai
        ├─────────────────────────────────────────────┤
        │  LỚP 1: MLOps core (data/model/serve/monitor)│  <- XÂY TRƯỚC (lõi)
        ├─────────────────────────────────────────────┤
        │  NỀN: DevOps của bạn (K8s, cloud, CI/CD, IaC)│  <- ĐÃ CÓ
        └─────────────────────────────────────────────┘
```
Vì sao thứ tự này: SRE và DevSecOps **áp lên một hệ ML đã tồn tại** — phải có hệ ML (lớp 1) trước thì mới có cái để làm reliable & secure. Đừng học song song cả ba từ đầu → loãng.

---

## Phần 3 — LỚP 1: Trở thành MLOps (lõi)

Mục tiêu: tự dựng & vận hành được pipeline ML end-to-end. Đây là phần chính, ~3-4 tháng.

### Bước 1.1 — Ngôn ngữ chung với Data Scientist (2-4 tuần)
> Bạn KHÔNG cần thành DS. Chỉ cần hiểu đủ để nói chuyện & vận hành cho họ.

Cần bổ sung kiến thức:
```text
- Vòng đời ML: train / validation / test split, training loop, inference.
- Khái niệm: overfitting/underfitting, bias-variance, hyperparameter, epoch.
- Metric: accuracy/precision/recall/F1 (phân loại), RMSE/MAE (hồi quy), AUC.
- Feature: feature engineering là gì, vì sao train-serving skew nguy hiểm.
- Python data stack đủ đọc hiểu: numpy, pandas, một chút scikit-learn/pytorch.
```
Việc cần làm: tự train 1 model đơn giản (sklearn trên dataset công khai) end-to-end 1 lần — để **thấy DS làm gì**, không phải để giỏi train.

### Bước 1.2 — Reproducibility & Experiment tracking (2 tuần)
Cần bổ sung: khái niệm tái lập (seed, môi trường, data version).
```text
- MLflow: track experiment (param/metric/artifact) + Model Registry (staging→prod).
- Data versioning: DVC hoặc lakeFS — version data như version code.
- Đóng gói môi trường: container hoá training/inference (bạn làm được).
```
Việc cần làm: bọc model ở bước 1.1 bằng MLflow tracking + đăng ký vào registry.

### Bước 1.3 — Model Serving (3-4 tuần) — nền DevOps của bạn tỏa sáng
```text
- Đóng gói model thành API: BentoML hoặc FastAPI + container -> deploy K8s (bạn quen).
- Serving framework: KServe hoặc Seldon Core (autoscale, canary, scale-to-zero).
- Online vs Batch inference: khi nào dùng cái nào.
- GPU trên K8s: nvidia device plugin, chia sẻ GPU (MIG/time-slicing), node pool GPU.
- Tối ưu inference: ONNX/TensorRT/quantization để giảm latency & cost.
```
Việc cần làm: deploy model qua KServe trên K8s, có autoscale + canary. **Đây là năng lực khác biệt của bạn** (DS thường không làm được phần này).

### Bước 1.4 — Pipeline & Orchestration (3 tuần)
```text
- Kubeflow Pipelines / Argo Workflows / Metaflow: tự động hoá train pipeline.
- Feature Store (Feast): đồng nhất feature giữa train và serve.
- CI/CD/CT: git push -> test data+model -> train -> EVAL GATE (metric đạt ngưỡng mới cho qua)
  -> register -> deploy. (Đây là CI/CD bạn quen + gate theo metric ML.)
```
Việc cần làm: dựng pipeline train→eval→register→deploy tự động, có eval gate.

### Bước 1.5 — ML Monitoring (2-3 tuần)
```text
- Infra monitoring (bạn đã có): latency, throughput, GPU util, error rate.
- ML monitoring (mới): data drift, concept drift, training-serving skew, model perf decay.
- Công cụ: Evidently (drift), Prometheus/Grafana (infra), alert -> trigger retrain.
```
Việc cần làm: gắn Evidently theo dõi drift + alert → nối vào pipeline retrain.

### (Tùy chọn) Bước 1.6 — Rẽ nhánh LLMOps nếu bạn làm LLM (4-6 tuần)
```text
- RAG end-to-end: chunking -> embedding -> vector DB (pgvector/Weaviate/Milvus) -> retrieval -> LLM.
- Serving LLM open-weight: vLLM/TGI, tối ưu KV-cache, batching, GPU memory.
- Prompt management: version prompt, test prompt như test code.
- Evaluation: eval set, LLM-as-judge, human feedback; đo hallucination.
- Guardrails: chống prompt injection, lọc output, PII redaction.
- Observability LLM: Langfuse/LangSmith/Phoenix — trace prompt→response, token, cost.
- Cost control: caching, model routing (rẻ↔mạnh), streaming.
```

### ✅ Cột mốc "đã là MLOps"
```text
[ ] Dựng được pipeline ML end-to-end: data(versioned) -> train -> eval gate -> registry -> serve -> monitor -> retrain.
[ ] Deploy model có autoscale/canary/rollback trên K8s (kể cả GPU).
[ ] Theo dõi được drift + trigger retrain tự động.
[ ] Nói chuyện trôi chảy với Data Scientist về vòng đời & metric.
[ ] (nếu LLM) dựng được RAG + serve LLM + eval + guardrails.
Capstone: 1 ML platform hoàn chỉnh bằng IaC + GitOps.
```

---

## Phần 4 — LỚP 2: Bồi SRE cho hệ ML

Mục tiêu: hệ ML không chỉ "chạy" mà **đáng tin ở quy mô**. ~1.5-2 tháng, làm SAU khi có lớp 1.

Cần bổ sung (áp lên chính hệ ML vừa xây):
```text
1. Định lượng reliability cho ML:
   - SLI/SLO cho ML: không chỉ latency/uptime mà cả CHẤT LƯỢNG (accuracy/quality threshold).
   - Error budget: khi model degrade quá ngưỡng -> "hết budget" -> freeze/rollback.
   - "Drift = incident": coi suy giảm chất lượng model như một sự cố production.

2. Serving reliability ở quy mô:
   - Capacity planning cho GPU (đắt -> autoscale/scale-to-zero thông minh, batching).
   - Xử lý cold start model lớn, timeout inference, hàng đợi request.
   - Load test hệ serving (k6/locust) -> tìm bottleneck.

3. Incident & vận hành:
   - Runbook cho sự cố ML (model xấu, drift, GPU hết, pipeline fail).
   - Rollback model nhanh (version registry -> serve version cũ).
   - Postmortem blameless; on-call cho hệ ML.
   - Chaos: thử kill serving pod/GPU node -> hệ tự hồi phục?

4. Coding tốt hơn: viết tool/automation/operator (SRE thiên software), không chỉ script.
```
Việc cần làm: đặt SLO cho serving (vd p99 < 300ms VÀ quality > ngưỡng), dựng dashboard error budget, viết runbook + chạy 1 game day.

### ✅ Cột mốc "SRE cho ML"
```text
[ ] Có SLO (latency + chất lượng) + error budget cho hệ ML, dashboard theo dõi.
[ ] Rollback model & xử lý drift-as-incident thành thạo, có runbook.
[ ] Tối ưu được GPU capacity/cost với autoscale.
[ ] Chạy được chaos/game day, hệ tự hồi phục.
```

---

## Phần 5 — LỚP 3: Bồi DevSecOps / MLSecOps

Mục tiêu: hệ ML **an toàn & tuân thủ**. ~1.5-2 tháng, làm sau cùng (áp lên hệ đã reliable).

Cần bổ sung:
```text
1. Nền security (nếu chưa chắc):
   - OWASP Top 10, threat modeling (STRIDE), least privilege, zero trust, assume breach.

2. Nhúng security vào pipeline ML (shift-left) — dùng CI/CD bạn đã có:
   - SAST (Semgrep) + SCA/CVE (Trivy) + secret scanning + IaC scan (checkov) -> fail build khi nghiêm trọng.
   - Supply chain: SBOM (Syft), ký image (cosign), admission control chỉ chạy image tin cậy.
   - Policy as Code: Kyverno/OPA enforce (bạn có nền k8s security).

3. MLSecOps — bảo mật ĐẶC THÙ ML (ngách hiếm):
   - Model supply chain: model tải từ HuggingFace có an toàn? (pickle có thể chứa code độc)
     -> dùng safetensors, quét & verify nguồn model.
   - Data governance: training data có PII? có bị data poisoning? lineage/audit để truy vết.
   - Model theft/extraction: bảo vệ model qua API.

4. LLM security (nếu làm LLM) — OWASP Top 10 for LLM:
   - Prompt injection, insecure output handling, data/context leakage, excessive agency (agent quyền quá lớn).
   - Guardrails: lọc input/output, chống jailbreak, PII redaction.

5. Compliance & governance AI:
   - Map hệ thống với framework (SOC2/CIS/GDPR); EU AI Act, model card, audit quyết định AI.
```
Việc cần làm: nhúng bộ scan vào CI (fail khi CVE nghiêm trọng), ký & verify image, quét model trước khi deploy, thêm guardrails cho LLM (nếu có), viết 1 policy Kyverno.

### ✅ Cột mốc "MLSecOps"
```text
[ ] Pipeline ML có security gate tự động (SAST/SCA/secret/IaC scan + image signing).
[ ] Model được quét & verify nguồn trước khi deploy (chống supply-chain).
[ ] Data governance: biết training data có gì, lineage, xử lý PII.
[ ] (nếu LLM) có guardrails chống prompt injection + kiểm soát output.
[ ] Hệ thống map được với ít nhất 1 framework tuân thủ.
```

---

## Phần 6 — Tổng lộ trình thời gian (tham khảo)

```text
Tháng 1-4  : LỚP 1 MLOps core (+ rẽ LLMOps nếu làm LLM)   -> đã là MLOps Engineer
Tháng 5-6  : LỚP 2 SRE cho ML                              -> ML Reliability
Tháng 7-8  : LỚP 3 MLSecOps                                -> AI Platform Engineer (full)
Xuyên suốt : viết ADR/design doc cho mỗi quyết định (dùng templates/), xây capstone công khai.
```
> Không cứng nhắc — tùy thời gian bạn có & dự án thực tế. Ưu tiên **học gắn với việc thật đang làm** (nhanh gấp đôi học chay).

## Phần 7 — Định hình vị thế cuối
```text
Nền DevOps (đã có) + MLOps (lõi) + SRE (tin cậy) + DevSecOps (an toàn)
   = ML/AI Platform Engineer / ML Infrastructure Architect
   -> xây & vận hành platform AI end-to-end: đáng tin, an toàn, tối ưu chi phí.
   -> hồ sơ IC/Architect cực hiếm, đúng làn sóng AI, thị trường thiếu trầm trọng.
```
Ba việc đòn bẩy nhất, làm xuyên suốt: **(1) học gắn dự án AI thật đang làm, (2) viết ADR/design doc mỗi quyết định, (3) xây 1 AI platform capstone công khai làm bằng chứng năng lực.**

## Tài nguyên
- MLOps: *Designing Machine Learning Systems* (Chip Huyen), *Machine Learning Engineering* (Andriy Burkov), Google "MLOps: Continuous delivery..." whitepaper, "Hidden Technical Debt in ML Systems".
- LLMOps: docs của LangChain/LlamaIndex, vLLM, Langfuse; "Building LLM apps" (Chip Huyen).
- SRE: *SRE Book* + *SRE Workbook* (Google, free).
- Security: OWASP Top 10 + **OWASP Top 10 for LLM Applications**, SLSA framework.
