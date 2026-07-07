# Fine-tuning & LoRA — chi tiết kỹ thuật (mức 3 của tùy biến LLM)

> Hoàn thiện bộ ba tùy biến LLM: **Prompt → RAG → Fine-tune**. File này giải thích fine-tuning là gì,
> **khi nào NÊN & khi nào KHÔNG**, các kỹ thuật (full FT vs LoRA/QLoRA), hạ tầng GPU, pipeline MLOps,
> và cách vận hành nhiều adapter ở production. Góc nhìn Platform Engineer, không phải nhà nghiên cứu.

## 📑 Nội dung
- [0. Quyết định TRƯỚC TIÊN: có nên fine-tune không?](#0-quyết-định-trước-tiên-có-nên-fine-tune-không)
- [1. Fine-tuning là gì & các loại](#1-fine-tuning-là-gì--các-loại)
- [2. LoRA & QLoRA — vì sao đổi cuộc chơi](#2-lora--qlora--vì-sao-đổi-cuộc-chơi)
- [3. Chuẩn bị dữ liệu (quyết định thành bại)](#3-chuẩn-bị-dữ-liệu-quyết-định-thành-bại)
- [4. Hạ tầng & GPU cho training](#4-hạ-tầng--gpu-cho-training)
- [5. Pipeline fine-tuning (MLOps)](#5-pipeline-fine-tuning-mlops)
- [6. Serving model đã fine-tune (nhiều adapter)](#6-serving-model-đã-fine-tune-nhiều-adapter)
- [7. Đánh giá & rủi ro](#7-đánh-giá--rủi-ro)
- [8. Cây quyết định & checklist](#8-cây-quyết-định--checklist)

---

## 0. Quyết định TRƯỚC TIÊN: có nên fine-tune không?

**Đây là phần quan trọng nhất.** Fine-tune đắt, phức tạp, dễ sai — đa số trường hợp **không cần**.

```text
Fine-tune KHÔNG dạy được KIẾN THỨC MỚI hiệu quả (đó là việc của RAG).
Fine-tune dạy được HÀNH VI / PHONG CÁCH / ĐỊNH DẠNG / KỸ NĂNG.
```

| Nhu cầu | Giải pháp đúng |
|---------|----------------|
| Thêm kiến thức riêng, dữ liệu cập nhật | **RAG** (không phải fine-tune) |
| Trả lời đúng format/schema cố định | Thử **prompt** trước → fine-tune nếu vẫn lỗi |
| Đổi giọng điệu/phong cách nhất quán | Prompt → fine-tune nếu cần ổn định cao |
| Kỹ năng chuyên biệt (vd phân loại domain hẹp, ngôn ngữ hiếm) | **Fine-tune** hợp lý |
| Giảm chi phí (model nhỏ fine-tune thay model lớn qua prompt dài) | **Fine-tune** hợp lý ở volume lớn |
| Model làm theo tool/format riêng ổn định | **Fine-tune** hợp lý |

**Quy tắc vàng:** `Prompt engineering → RAG → (chỉ khi hai cái trên không đạt) → Fine-tune.`
Lý do: prompt/RAG rẻ, nhanh, dễ đổi. Fine-tune tốn GPU, cần data chất lượng, khó cập nhật, và **model có thể "quên"** kiến thức cũ (catastrophic forgetting).

---

## 1. Fine-tuning là gì & các loại

Fine-tuning = huấn luyện tiếp một model nền (pretrained) trên data của bạn để chỉnh trọng số.

```text
Các mức "training" LLM:
  Pre-training       : train từ đầu trên hàng nghìn tỷ token (chỉ big lab làm, cực đắt).
  Continued pre-train: học tiếp trên domain corpus lớn (ít gặp, tốn kém).
  Fine-tuning (SFT)  : Supervised Fine-Tuning trên cặp (input -> output mong muốn). PHỔ BIẾN NHẤT.
  Preference tuning  : RLHF / DPO — căn chỉnh theo sở thích con người (nâng cao).
```

**Full fine-tuning vs Parameter-Efficient (PEFT):**
```text
Full FT   : cập nhật TẤT CẢ trọng số. Cần GPU khổng lồ (model 7B cần ~vài chục-trăm GB VRAM
            cho weight + gradient + optimizer state). Đắt, mỗi task 1 bản model đầy đủ.
PEFT/LoRA : chỉ train một phần NHỎ trọng số thêm vào. Rẻ hơn 10-100x, 1 base + nhiều adapter nhỏ.
            -> Đây là cách gần như mọi team dùng hiện nay.
```

---

## 2. LoRA & QLoRA — vì sao đổi cuộc chơi

### LoRA (Low-Rank Adaptation)
```text
Ý tưởng: KHÔNG sửa trọng số gốc (đóng băng). Thêm vào mỗi lớp một cặp ma trận nhỏ (rank thấp A,B)
         -> chỉ train A,B (rất ít tham số, ~0.1-1% model).
Kết quả: - Base model giữ nguyên -> nhiều task chỉ cần nhiều "adapter" nhỏ (vài MB-vài trăm MB).
         - Train nhanh, ít VRAM hơn nhiều.
         - Serve: nạp 1 base + gắn/tháo adapter theo task (thậm chí nhiều adapter cùng lúc).
```
Tham số quan trọng: `r` (rank — độ lớn adapter, thường 8-64), `alpha` (scaling), `target_modules` (lớp nào gắn LoRA — thường attention q/k/v/o), `dropout`.

### QLoRA (Quantized LoRA) — chạy được trên GPU nhỏ
```text
Base model được QUANTIZE xuống 4-bit (giảm VRAM mạnh) + LoRA adapter train ở trên.
-> Fine-tune model 7B-13B trên MỘT GPU consumer (24GB), thậm chí model lớn hơn.
Đánh đổi: chất lượng gần full LoRA, chậm hơn chút. Là lựa chọn thực tế cho team nhỏ/ngân sách hạn chế.
```

**So sánh nhanh:**
| | Full FT | LoRA | QLoRA |
|--|---------|------|-------|
| VRAM cần | Rất cao | Trung bình | Thấp (chạy GPU nhỏ) |
| Tốc độ train | Chậm | Nhanh | Nhanh (hơi chậm hơn LoRA) |
| Chất lượng | Cao nhất | Rất tốt (đủ đa số case) | Tốt |
| Output | 1 model đầy đủ/task | base + adapter nhỏ | base(4bit) + adapter |
| Multi-task serving | Tốn (nhiều model) | Tuyệt (nhiều adapter/1 base) | Tuyệt |

> Với vai trò Platform Engineer: **LoRA/QLoRA là mặc định.** Full FT chỉ khi có lý do rõ + đủ GPU.

---

## 3. Chuẩn bị dữ liệu (quyết định thành bại)

**Sự thật:** chất lượng data quan trọng hơn số lượng và hơn cả thuật toán. Data rác → model rác.

```text
- Định dạng SFT phổ biến: cặp instruction/response (hoặc chat format nhiều lượt).
    {"messages": [{"role":"system","content":"..."},
                  {"role":"user","content":"..."},
                  {"role":"assistant","content":"<output mong muốn>"}]}
- Số lượng: vài trăm-vài nghìn ví dụ CHẤT LƯỢNG CAO thường tốt hơn hàng chục nghìn ví dụ rác.
- Tính nhất quán: format/giọng điệu đồng nhất -> model học được pattern rõ.
- Đa dạng: bao phủ các trường hợp thật, kể cả edge case.
- Tách train/validation: giữ tập validation để đo overfitting.
- Làm sạch: bỏ trùng lặp, sai nhãn, PII (trừ khi cố ý), nội dung độc hại.
```
**Data versioning (MLOps):** dataset fine-tune phải versioned (DVC/lakeFS) — vì kết quả model phụ thuộc trực tiếp vào data version. Không tái lập được data = không tái lập được model.

**Trách nhiệm Platform Engineer ở đây:** dựng pipeline data (thu thập → làm sạch → validate schema → version), không nhất thiết tự tạo nội dung (việc của domain expert/DS), nhưng đảm bảo **quy trình & chất lượng & truy vết**.

---

## 4. Hạ tầng & GPU cho training

```text
VRAM cần (ước lượng thô, model 7B):
  Full FT (FP16)     : ~ >100GB (weight + gradient + optimizer Adam) -> nhiều GPU / A100 80GB.
  LoRA (FP16 base)   : ~ 16-24GB -> 1 GPU tầm trung.
  QLoRA (4-bit base) : ~ 8-12GB -> 1 GPU consumer (RTX 3090/4090, A10G).
Model 13B/70B -> nhân lên; 70B thường cần multi-GPU + tensor/pipeline parallelism kể cả LoRA.
```

**Kỹ thuật giảm VRAM (biết để cấu hình):**
```text
- Gradient checkpointing: đổi compute lấy memory (chậm hơn, đỡ VRAM).
- Mixed precision (bf16/fp16): chuẩn.
- Gradient accumulation: batch nhỏ tích lũy -> giả lập batch lớn khi thiếu VRAM.
- Quantization (4-bit) cho base (QLoRA).
- DeepSpeed ZeRO / FSDP: chia optimizer state/gradient/param qua nhiều GPU (cho model lớn).
- Flash Attention: nhanh + tiết kiệm memory cho attention.
```

**Trên Kubernetes (training job):**
```yaml
apiVersion: batch/v1
kind: Job
metadata: { name: finetune-qlora, namespace: ml-train }
spec:
  backoffLimit: 1
  template:
    spec:
      restartPolicy: Never
      nodeSelector: { "nvidia.com/gpu.product": "A100-SXM4-80GB" }
      tolerations:
        - { key: "nvidia.com/gpu", operator: "Exists", effect: "NoSchedule" }
      containers:
        - name: trainer
          image: registry.example.com/finetune:1.2.0   # chứa transformers/peft/trl/bitsandbytes
          command: ["python", "train_qlora.py"]
          env:
            - { name: BASE_MODEL, value: "mistralai/Mistral-7B-v0.3" }
            - { name: DATASET_URI, value: "s3://ml-data/sft/v5/" }
            - { name: OUTPUT_URI,  value: "s3://ml-models/adapters/support-bot/" }
          resources:
            limits: { nvidia.com/gpu: "1", memory: 64Gi, cpu: "8" }
          volumeMounts: [{ name: shm, mountPath: /dev/shm }]
      volumes: [{ name: shm, emptyDir: { medium: Memory, sizeLimit: 16Gi } }]
```
- Job xong → adapter (nhỏ) đẩy lên S3/registry. Với training dài/nhiều GPU → dùng **Kubeflow Training Operator** (PyTorchJob) hoặc **Volcano**/Ray cho scheduling GPU tốt hơn.
- **Spot GPU** cho training chịu gián đoạn + **checkpoint định kỳ** để resume khi bị thu hồi → tiết kiệm lớn.

**Thư viện phổ biến:** `transformers` + `peft` (LoRA) + `trl` (SFTTrainer) + `bitsandbytes` (4-bit) + `accelerate`/`deepspeed`. Hoặc framework cao hơn: **Axolotl**, **Llama-Factory** (config YAML, ít code).

---

## 5. Pipeline fine-tuning (MLOps)

```text
[Data thu thập] -> [Làm sạch + validate + version (DVC)] -> [SFT/LoRA train (Job GPU)]
   -> [Eval trên validation + eval set nghiệp vụ] -> [Eval GATE: đạt ngưỡng?]
      -> [Đăng ký adapter vào registry (MLflow) + metadata: base model, data version, hyperparam]
         -> [Deploy serving (canary)] -> [Monitor] -> (data mới -> lặp lại)
```
Nguyên tắc reproducibility (bắt buộc):
```text
Ghi lại đủ để TÁI LẬP: base model + version, dataset version, hyperparameter (r/alpha/lr/epochs),
seed, môi trường (image). Thiếu 1 cái -> không tái tạo được kết quả -> không debug được khi lỗi.
```
Đây chính là chỗ nền MLOps (experiment tracking + data versioning + registry) của bạn áp vào.

---

## 6. Serving model đã fine-tune (nhiều adapter)

Đây là lợi thế vận hành lớn của LoRA — và là việc của Platform Engineer:

```text
Cách 1 — Merge adapter vào base:
  gộp adapter + base -> 1 model đầy đủ -> serve như model thường (vLLM).
  Ưu: đơn giản, nhanh nhất khi infer. Nhược: mỗi task 1 model to -> tốn VRAM nếu nhiều task.

Cách 2 — Multi-LoRA serving (KHÔNG merge):
  nạp 1 BASE model + nhiều adapter nhỏ, chọn adapter theo request.
  vLLM hỗ trợ Multi-LoRA -> phục vụ nhiều "model tùy biến" trên 1 base, tiết kiệm GPU cực lớn.
  Ưu: 1 GPU phục vụ N task. Nhược: cấu hình phức tạp hơn, overhead nhỏ.
```
Ví dụ ý tưởng vLLM multi-LoRA:
```bash
# vLLM nạp base + nhiều adapter, route theo tên model trong request
python -m vllm.entrypoints.openai.api_server \
  --model mistralai/Mistral-7B-v0.3 \
  --enable-lora \
  --lora-modules support-bot=s3://.../support/ sales-bot=s3://.../sales/
# Request chỉ định "model": "support-bot" -> dùng adapter tương ứng.
```
> **Điểm ghi điểm:** biết multi-LoRA serving = phục vụ nhiều sản phẩm AI trên cùng hạ tầng GPU → tiết kiệm chi phí lớn. Đây là kiến thức hiếm.

Phần còn lại (autoscale, canary, GPU, observability) giống [ml-serving-k8s.md](./ml-serving-k8s.md) và [llmops-end-to-end.md](./llmops-end-to-end.md).

---

## 7. Đánh giá & rủi ro

**Đánh giá:** dùng eval set nghiệp vụ (như [llmops-end-to-end.md](./llmops-end-to-end.md) mục 3) — so model fine-tuned vs base vs prompt/RAG. Đừng chỉ nhìn training loss (loss thấp ≠ tốt cho task thật).

**Rủi ro đặc thù fine-tuning:**
```text
- Catastrophic forgetting: model "quên" kỹ năng chung khi học task hẹp -> giữ data đa dạng,
  LoRA (đóng băng base) giảm rủi ro này so với full FT.
- Overfitting: học thuộc data train, kém trên input mới -> validation set + early stopping, ít epoch.
- Data leakage/PII: data train lọt vào output -> làm sạch data, guardrails output.
- Bias khuếch đại: data lệch -> model lệch -> kiểm tra fairness.
- Chi phí ẩn: fine-tune tạo nợ bảo trì (base model mới ra -> phải fine-tune lại).
- Khó cập nhật kiến thức: kiến thức thay đổi -> RAG cập nhật dễ, fine-tune phải train lại.
```

---

## 8. Cây quyết định & checklist

```text
Cần thêm KIẾN THỨC/dữ liệu cập nhật?           -> RAG (đừng fine-tune)
Prompt engineering đã thử kỹ chưa?             -> thử trước
Cần HÀNH VI/FORMAT/PHONG CÁCH ổn định mà       -> Fine-tune (LoRA/QLoRA)
  prompt+RAG không đạt?
Ngân sách GPU hạn chế?                          -> QLoRA
Nhiều task/sản phẩm trên cùng base?            -> LoRA + Multi-LoRA serving
Cần chất lượng tối đa + có nhiều GPU?          -> cân nhắc Full FT (hiếm khi cần)
```

**Checklist fine-tuning production:**
```text
[ ] Đã xác nhận RAG/prompt KHÔNG đủ (không fine-tune vô cớ).
[ ] Data chất lượng cao, nhất quán, đã làm sạch + version (DVC).
[ ] Chọn LoRA/QLoRA (mặc định) trừ khi có lý do full FT.
[ ] Training job trên GPU (K8s Job/Kubeflow), checkpoint + Spot nếu chịu gián đoạn.
[ ] Ghi đủ để tái lập: base+version, data version, hyperparam, seed, image.
[ ] Eval GATE bằng eval set nghiệp vụ (so với base) trước khi deploy.
[ ] Adapter đăng ký registry + metadata đầy đủ.
[ ] Serving: merge hoặc Multi-LoRA (tiết kiệm GPU), canary + rollback.
[ ] Monitor chất lượng online + guardrails; kế hoạch retrain khi base/data đổi.
```

## Tóm tắt
- Fine-tune dạy **hành vi/format/kỹ năng**, KHÔNG phải kiến thức (đó là RAG). Thứ tự: prompt → RAG → fine-tune.
- **LoRA/QLoRA** là mặc định: rẻ, 1 base + nhiều adapter nhỏ; QLoRA chạy được trên GPU nhỏ.
- Data chất lượng + versioning quyết định thành bại; reproducibility bắt buộc.
- **Multi-LoRA serving** (vLLM) = phục vụ nhiều sản phẩm AI trên 1 base → tiết kiệm GPU lớn (điểm khác biệt).
- Cảnh giác: catastrophic forgetting, overfitting, nợ bảo trì khi base model mới ra.
