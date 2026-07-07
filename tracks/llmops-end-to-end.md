# LLMOps End-to-End — RAG + Serving vLLM + Eval + Guardrails (chi tiết kỹ thuật)

> Hướng dẫn kỹ thuật để xây & vận hành ứng dụng LLM production. Bốn trụ cột:
> **RAG** (đưa kiến thức riêng vào LLM), **Serving** (self-host LLM hiệu quả), **Eval** (đo chất lượng),
> **Guardrails** (an toàn). Kèm kiến trúc, config, và quyết định kỹ thuật.

## 📑 Nội dung
- [0. Kiến trúc tổng thể](#0-kiến-trúc-tổng-thể)
- [1. RAG — Retrieval-Augmented Generation](#1-rag--retrieval-augmented-generation)
- [2. Serving LLM với vLLM](#2-serving-llm-với-vllm)
- [3. Evaluation — đo chất lượng LLM](#3-evaluation--đo-chất-lượng-llm)
- [4. Guardrails — an toàn](#4-guardrails--an-toàn)
- [5. Observability & Cost](#5-observability--cost)
- [6. CI/CD cho LLM app](#6-cicd-cho-llm-app)
- [7. Quyết định kiến trúc hay gặp](#7-quyết-định-kiến-trúc-hay-gặp)

---

## 0. Kiến trúc tổng thể

```text
                        ┌─────────── Guardrails (input) ───────────┐
User query ──> App/Orchestrator (LangChain/LlamaIndex/custom)
                   │
                   ├─(RAG)─> Embed query -> Vector DB -> top-k chunks -> ghép vào prompt
                   │
                   ├─> LLM (self-host vLLM  |  API Claude/OpenAI)
                   │
                   └─(Guardrails output)-> lọc/kiểm tra -> response -> User
                   │
        Observability (Langfuse/Phoenix): trace prompt->retrieval->response, token, cost, latency
        Eval (offline + online): chất lượng, hallucination, relevance
```

**3 mức tùy biến LLM (chọn theo nhu cầu, tăng dần độ phức tạp):**
```text
1. Prompt engineering  -> rẻ nhất, nhanh nhất. Thử TRƯỚC.
2. RAG                 -> khi cần kiến thức riêng/cập nhật (docs công ty, dữ liệu mới).
3. Fine-tuning (LoRA)  -> khi cần thay đổi HÀNH VI/phong cách/format mà prompt+RAG không đạt.
Nguyên tắc: prompt -> RAG -> fine-tune. Đừng fine-tune khi RAG chưa thử.
```

---

## 1. RAG — Retrieval-Augmented Generation

Cho LLM trả lời dựa trên **kiến thức riêng** của bạn thay vì chỉ kiến thức training. Giảm hallucination, cập nhật được, trích nguồn.

### Pipeline RAG (2 giai đoạn)
```text
INDEXING (offline, chạy khi data đổi):
  Documents -> Chunking (chia đoạn) -> Embedding (vector hoá) -> lưu Vector DB

RETRIEVAL + GENERATION (online, mỗi query):
  Query -> Embedding -> tìm top-k chunk gần nhất (vector search)
        -> ghép chunk vào prompt (context) -> LLM sinh câu trả lời (kèm nguồn)
```

### Các quyết định kỹ thuật quan trọng
| Thành phần | Lựa chọn & lưu ý |
|-----------|------------------|
| **Chunking** | kích thước chunk (256-1024 token), overlap (~10-20%). Chunk quá to → nhiễu; quá nhỏ → mất ngữ cảnh. Thử nghiệm theo loại tài liệu. |
| **Embedding model** | chất lượng embedding quyết định retrieval. Chọn theo ngôn ngữ (có tiếng Việt không), độ dài, chi phí. Nhất quán model giữa index & query. |
| **Vector DB** | pgvector (đơn giản, dùng Postgres sẵn có), Qdrant/Weaviate/Milvus (chuyên dụng, scale), Pinecone (managed). |
| **Retrieval** | top-k (thường 3-8), similarity (cosine). Nâng cao: **hybrid search** (vector + keyword/BM25), **reranking** (cross-encoder xếp lại top-k → tăng độ chính xác). |
| **Prompt template** | cách ghép context + hướng dẫn "chỉ trả lời dựa trên context, không bịa, trích nguồn". |

### Vector DB trên K8s — ví dụ pgvector (đơn giản, tận dụng Postgres)
```sql
CREATE EXTENSION vector;
CREATE TABLE docs (id bigserial, content text, embedding vector(1024));
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops);   -- index nhanh
-- Retrieval: tìm 5 chunk gần nhất
SELECT content FROM docs ORDER BY embedding <=> :query_embedding LIMIT 5;
```

### Cải thiện chất lượng RAG (khi câu trả lời kém)
```text
- Retrieval kém (lấy sai chunk)  -> đổi embedding model, thêm hybrid search + reranking, chỉnh chunking.
- Context đúng nhưng trả lời sai -> chỉnh prompt (bắt bám context, cấm bịa), tăng/giảm k.
- Không đủ thông tin            -> mở rộng knowledge base, chunking tốt hơn.
- Đánh giá RAG: đo cả RETRIEVAL (chunk lấy về có liên quan?) và GENERATION (trả lời có đúng/bám nguồn?).
```

---

## 2. Serving LLM với vLLM

Khi self-host model open-weight (Llama, Mistral, Qwen...), **vLLM** là runtime hiệu năng cao (PagedAttention, continuous batching → throughput cao hơn nhiều so với chạy thô).

### Vì sao vLLM (so với chạy transformers thô)
```text
- Continuous batching: gom request liên tục -> tận dụng GPU tối đa (throughput cao).
- PagedAttention: quản lý KV-cache như paging -> tiết kiệm GPU memory, phục vụ nhiều request hơn.
- OpenAI-compatible API -> đổi từ OpenAI/Claude API sang self-host ít sửa code.
- Hỗ trợ quantization (AWQ/GPTQ), tensor parallelism (chia model qua nhiều GPU).
```

### Deploy vLLM trên K8s (GPU)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata: { name: vllm-mistral, namespace: llm-prod }
spec:
  replicas: 1
  selector: { matchLabels: { app: vllm-mistral } }
  template:
    metadata: { labels: { app: vllm-mistral } }
    spec:
      nodeSelector: { "nvidia.com/gpu.product": "A100-SXM4-40GB" }
      containers:
        - name: vllm
          image: vllm/vllm-openai:latest
          args:
            - "--model=mistralai/Mistral-7B-Instruct-v0.3"
            - "--gpu-memory-utilization=0.90"   # dùng 90% VRAM cho KV-cache
            - "--max-model-len=8192"
            - "--dtype=float16"
            # - "--quantization=awq"            # nếu dùng model quantized (giảm VRAM)
            # - "--tensor-parallel-size=2"      # chia model qua 2 GPU nếu model to
          ports: [{ containerPort: 8000 }]
          resources:
            limits: { nvidia.com/gpu: "1", memory: 32Gi }
          readinessProbe:                       # model nạp xong mới nhận traffic
            httpGet: { path: /health, port: 8000 }
            initialDelaySeconds: 60
            periodSeconds: 10
```
```bash
# Gọi (OpenAI-compatible):
curl http://vllm-mistral:8000/v1/chat/completions -H "Content-Type: application/json" -d '{
  "model": "mistralai/Mistral-7B-Instruct-v0.3",
  "messages": [{"role":"user","content":"Xin chào"}]
}'
```

### Quyết định serving quan trọng
```text
- GPU memory: model 7B FP16 ~ 14GB weight + KV-cache. Model to hơn -> quantize (AWQ/GPTQ)
  hoặc tensor parallelism qua nhiều GPU.
- Throughput vs latency: batch lớn -> throughput cao nhưng latency/request tăng. Cân theo use case
  (chatbot cần latency thấp; xử lý hàng loạt ưu tiên throughput).
- Streaming: trả token dần (SSE) -> UX tốt hơn cho chat.
- Autoscale: LLM cold start LÂU (nạp GB weight) -> thường giữ minReplicas>=1, scale bằng KEDA theo
  queue/GPU. Scale-to-zero chỉ cho môi trường dev.
- API vs self-host: dùng API (Claude/OpenAI) khi tải thấp/không muốn quản GPU; self-host khi
  volume lớn (rẻ hơn ở quy mô), cần riêng tư dữ liệu, hoặc cần model tùy biến.
```

---

## 3. Evaluation — đo chất lượng LLM (khó nhất)

Khác ML cổ điển (có accuracy rõ), LLM **không có 1 đáp án đúng** → eval là thách thức riêng.

### Các phương pháp (kết hợp nhiều)
```text
1. Eval set thủ công: bộ câu hỏi + đáp án/tiêu chí kỳ vọng -> chạy hồi quy mỗi khi đổi prompt/model.
2. LLM-as-judge: dùng 1 LLM mạnh chấm output theo tiêu chí (relevance, correctness, tone).
   -> nhanh, scale được, nhưng phải kiểm chứng judge đáng tin.
3. Metric tự động: cho RAG -> faithfulness (bám nguồn?), answer relevance, context precision/recall
   (RAGAS). Cho task có tham chiếu -> BLEU/ROUGE/semantic similarity.
4. Human eval: con người chấm mẫu -> chuẩn vàng nhưng đắt/chậm. Dùng để hiệu chỉnh LLM-judge.
5. Online eval: thu thập feedback thật (thumbs up/down), đo tỷ lệ user hài lòng.
```

### Đo cái gì
| Chiều | Câu hỏi |
|-------|---------|
| Correctness / Faithfulness | Trả lời đúng & bám nguồn (không bịa)? |
| Relevance | Có trả lời đúng câu hỏi? |
| Retrieval quality (RAG) | Chunk lấy về có liên quan? |
| Safety | Có nội dung độc hại/PII/lệch chuẩn? |
| Latency / Cost | Nhanh & rẻ đủ chưa? |

### Công cụ eval
```text
- RAGAS: metric chuyên cho RAG (faithfulness, relevance, context...).
- DeepEval / promptfoo: eval framework, chạy trong CI (regression test cho prompt/model).
- Langfuse / LangSmith / Phoenix: eval online + trace + dataset.
```
> **Nguyên tắc:** coi prompt & model như code → mỗi thay đổi phải **chạy eval set (regression)** trước khi lên prod. Đây là "CI cho LLM".

---

## 4. Guardrails — an toàn

LLM có rủi ro riêng (OWASP Top 10 for LLM). Guardrails = kiểm soát input & output.

```text
INPUT guardrails (trước khi tới LLM):
  - Prompt injection detection: chặn input cố ghi đè hướng dẫn ("bỏ qua chỉ thị trên...").
  - PII detection/redaction: che thông tin nhạy cảm.
  - Topic/policy filter: chặn câu hỏi ngoài phạm vi / vi phạm chính sách.
  - Rate limit theo user.

OUTPUT guardrails (sau khi LLM sinh):
  - Toxicity/safety filter: chặn nội dung độc hại.
  - PII leakage check: không để lộ dữ liệu nhạy cảm/context nội bộ.
  - Format/schema validation: ép output đúng JSON/schema (đặc biệt khi output làm đầu vào hệ khác).
  - Grounding check: output có bám context RAG không (chống bịa).
  - Insecure output handling: KHÔNG chạy trực tiếp output LLM như code/SQL/lệnh.
```

### Rủi ro then chốt (OWASP LLM) & cách giảm
| Rủi ro | Mô tả | Giảm thiểu |
|--------|-------|-----------|
| **Prompt injection** | input chèn lệnh điều khiển LLM | tách rõ system/user prompt, phát hiện injection, least privilege cho tool |
| **Insecure output handling** | output LLM chạy như code/SQL | không eval/exec output; validate & escape |
| **Sensitive info disclosure** | lộ data training/context | PII redaction, output filter, phân quyền context |
| **Excessive agency** | LLM-agent có quá nhiều quyền hành động | giới hạn tool, human-in-the-loop cho hành động nguy hiểm |
| **Data poisoning** | data huấn luyện/RAG bị đầu độc | kiểm soát nguồn, verify data ingest |

### Công cụ
```text
- NeMo Guardrails, Guardrails AI, Llama Guard (model phân loại an toàn),
  presidio (PII), + logic tự viết trong orchestrator.
```

---

## 5. Observability & Cost

```text
Trace (bắt buộc cho LLM): mỗi request log toàn bộ chuỗi
  query -> retrieval (chunk nào) -> prompt cuối -> response -> token in/out -> latency -> cost.
  Công cụ: Langfuse (open-source, self-host được), LangSmith, Arize Phoenix.

Metrics:
  - Token usage & COST (theo user/feature) — LLM tính tiền theo token, dễ đội chi phí.
  - Latency (time-to-first-token quan trọng cho streaming, total latency).
  - Error/timeout rate, retrieval hit rate.
  - Chất lượng online (feedback, eval score).

Tối ưu COST (kỹ năng giá trị):
  - Semantic caching: cache câu hỏi tương tự -> không gọi LLM lại.
  - Model routing: câu dễ -> model nhỏ/rẻ; câu khó -> model mạnh.
  - Prompt tối ưu: giảm token thừa trong context (RAG lấy ít chunk hơn nhưng đúng hơn).
  - Self-host khi volume lớn (điểm hoà vốn so với API).
  - Giới hạn max_tokens output.
```

---

## 6. CI/CD cho LLM app

```text
Git push (đổi prompt / RAG config / model / code)
  -> Test: unit + EVAL SET regression (promptfoo/DeepEval) -> chặn nếu chất lượng tụt
  -> Build image -> scan (Trivy) -> ký (cosign)
  -> Deploy canary (10% traffic) -> theo dõi eval online + latency + cost
  -> 100% nếu ổn, rollback nếu xấu.
Versioning: prompt + model + retrieval index + config đều versioned (mọi thứ ảnh hưởng output).
```
> Điểm khác biệt với CI thường: **prompt/model là "code"** → phải có eval gate, không chỉ unit test.

---

## 7. Quyết định kiến trúc hay gặp

```text
API (Claude/OpenAI) vs Self-host (vLLM)?
  API: nhanh triển khai, không quản GPU, trả theo token. Hợp tải thấp/vừa, cần chất lượng cao nhất.
  Self-host: rẻ hơn ở VOLUME lớn, dữ liệu riêng tư, tùy biến model. Cần quản GPU + kỹ năng serving.
  -> Nhiều hệ dùng CẢ HAI: self-host cho phần lớn, fallback API cho câu khó.

Prompt vs RAG vs Fine-tune?
  -> Luôn thử theo thứ tự đó. Fine-tune chỉ khi cần đổi hành vi/format mà prompt+RAG không đạt.

Vector DB nào?
  -> pgvector nếu đã có Postgres & quy mô vừa. Qdrant/Milvus/Weaviate khi cần scale/tính năng.

Đánh giá "đủ tốt" thế nào?
  -> Định nghĩa eval set + ngưỡng chất lượng TRƯỚC, rồi mới tối ưu tới ngưỡng đó (tránh tối ưu vô hạn).
```

---

## Tóm tắt — checklist LLM app production
```text
[ ] RAG: chunking + embedding + vector DB + (hybrid + rerank nếu cần), prompt bám nguồn.
[ ] Serving: vLLM (nếu self-host) tối ưu GPU memory/batching, hoặc API có fallback.
[ ] Eval set + regression trong CI (prompt/model là code); online feedback.
[ ] Guardrails input & output (prompt injection, PII, toxicity, schema, grounding).
[ ] Observability: trace đầy đủ + token/cost + latency; alert.
[ ] Cost control: caching, model routing, max_tokens, tối ưu context.
[ ] Versioning: prompt + model + index + config.
[ ] Canary + rollback; manifest trong Git (GitOps).
```
