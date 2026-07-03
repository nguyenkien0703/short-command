# Vector Database ở quy mô lớn — HNSW, Sharding, Hybrid, Latency

> Vector search là trái tim của RAG & semantic search. File này đi sâu **cách hoạt động của ANN index
> (HNSW/IVF), tuning tham số, sharding/replication, hybrid search, và đánh đổi latency ↔ recall ↔ cost**
> khi lên hàng chục-hàng trăm triệu vector. Góc nhìn vận hành hạ tầng.

## 📑 Nội dung
- [0. Bài toán cốt lõi: ANN chứ không phải exact search](#0-bài-toán-cốt-lõi-ann-chứ-không-phải-exact-search)
- [1. HNSW — index phổ biến nhất](#1-hnsw--index-phổ-biến-nhất)
- [2. IVF & Quantization (khi RAM là ràng buộc)](#2-ivf--quantization-khi-ram-là-ràng-buộc)
- [3. Sharding & Replication (scale ngang)](#3-sharding--replication-scale-ngang)
- [4. Hybrid search & Filtering](#4-hybrid-search--filtering)
- [5. Latency ↔ Recall ↔ Cost](#5-latency--recall--cost-tam-giác-đánh-đổi)
- [6. Kiến trúc & vận hành ở quy mô](#6-kiến-trúc--vận-hành-ở-quy-mô)
- [7. Chọn vector DB](#7-chọn-vector-db)
- [8. Checklist](#8-checklist-vector-search-ở-quy-mô)

---

## 0. Bài toán cốt lõi: ANN chứ không phải exact search

```text
Cần: cho query vector, tìm k vector GẦN NHẤT trong N vector (theo cosine/dot/L2).
Exact (brute-force): so với tất cả N -> chính xác 100% nhưng O(N) -> KHÔNG scale (N=100 triệu là chết).
ANN (Approximate Nearest Neighbor): chấp nhận GẦN đúng để nhanh gấp hàng nghìn lần.
```
**Khái niệm then chốt — Recall:**
```text
Recall@k = (số vector đúng trong top-k ANN trả về) / (top-k thật sự).
Recall 0.95 = 95% kết quả khớp với brute-force. Đây là "độ chính xác" của ANN.
Mọi tuning vector DB là đánh đổi: Recall cao <-> Latency thấp <-> RAM/Cost thấp. Chọn 2, hi sinh 1.
```

---

## 1. HNSW — index phổ biến nhất

**HNSW (Hierarchical Navigable Small World)** = đồ thị nhiều tầng, tìm kiếm bằng cách "nhảy" từ node tới node gần hơn. Mặc định của hầu hết vector DB (pgvector, Qdrant, Weaviate, Milvus, Elasticsearch).

```text
Cấu trúc: nhiều tầng như "skip list" trên đồ thị.
  Tầng cao  : ít node, cạnh dài -> nhảy nhanh tới vùng đúng (như đường cao tốc).
  Tầng thấp : nhiều node, cạnh ngắn -> tìm chính xác lân cận (như đường nội bộ).
Search: bắt đầu tầng cao -> đi xuống dần -> ở tầng đáy tìm k gần nhất.
Đặc điểm: rất nhanh, recall cao, NHƯNG tốn RAM (giữ cả đồ thị trong bộ nhớ) và build chậm.
```

### 3 tham số phải hiểu (tuning HNSW)
| Tham số | Ảnh hưởng | Đánh đổi |
|---------|-----------|----------|
| **M** (số cạnh/node) | độ kết nối đồ thị | M cao → recall tốt hơn, **RAM & build time tăng**. Thường 16-64. |
| **ef_construction** | độ "kỹ" khi BUILD index | cao → index chất lượng hơn (recall tốt) nhưng build chậm hơn. Thường 100-500. |
| **ef_search** (ef) | độ "kỹ" khi TRUY VẤN | cao → recall cao hơn nhưng **latency tăng**. Chỉnh lúc query. Thường 50-400. |

```text
Quy tắc tuning thực chiến:
  - ef_search là "núm xoay" runtime: tăng để lấy recall, giảm để lấy latency. Chỉnh KHÔNG cần rebuild.
  - M và ef_construction quyết định lúc build -> đổi phải rebuild index (đắt).
  - Bắt đầu: M=16, ef_construction=200, ef_search=100. Đo recall & latency -> chỉnh ef_search trước.
  - Cần recall cao hơn mà latency vẫn ổn -> tăng ef_search. Vẫn không đủ -> tăng M (rebuild).
```

### Ví dụ pgvector
```sql
-- Build HNSW index
CREATE INDEX ON docs USING hnsw (embedding vector_cosine_ops)
  WITH (m = 16, ef_construction = 200);
-- Tuning lúc query (runtime)
SET hnsw.ef_search = 100;   -- tăng lên 200-400 nếu cần recall cao hơn
SELECT id FROM docs ORDER BY embedding <=> :q LIMIT 10;
```

---

## 2. IVF & Quantization (khi RAM là ràng buộc)

HNSW tốn RAM. Ở quy mô rất lớn (hàng trăm triệu+ vector), RAM = tiền. Có kỹ thuật giảm.

### IVF (Inverted File Index)
```text
Ý tưởng: clustering vector thành nlist "ô" (centroid). Query -> chỉ tìm trong nprobe ô gần nhất.
  nlist  : số cụm (nhiều -> mỗi cụm nhỏ, nhanh nhưng dễ miss). ~ sqrt(N).
  nprobe : số cụm quét khi query (cao -> recall tốt, chậm hơn). Núm xoay runtime.
Ưu: ít RAM hơn HNSW, build nhanh. Nhược: recall thường kém HNSW ở cùng latency.
Thường kết hợp: IVF + PQ (bên dưới).
```

### Quantization — nén vector để giảm RAM
| Kỹ thuật | Cơ chế | Đánh đổi |
|----------|--------|----------|
| **SQ (Scalar Quantization)** | float32 → int8 (giảm 4x RAM) | mất ít recall, đơn giản |
| **PQ (Product Quantization)** | chia vector thành đoạn, mã hoá bằng codebook (nén mạnh) | RAM giảm nhiều, recall giảm rõ hơn |
| **Binary quantization** | vector → bit (nén cực mạnh) | rất nhanh/nhẹ, recall giảm nhiều, hợp làm bước lọc thô + rerank |

```text
Chiến lược quy mô lớn phổ biến: index nén (IVF-PQ hoặc HNSW+SQ) để lọc nhanh + RAM thấp
  -> lấy top candidates -> RERANK bằng vector gốc (full precision) để phục hồi recall.
```

**Điểm vận hành:** ở N nhỏ-vừa (< vài chục triệu) → HNSW thuần thường tốt nhất & đơn giản. Chỉ khi RAM/cost thành vấn đề mới cần IVF/quantization.

---

## 3. Sharding & Replication (scale ngang)

Khi 1 node không đủ RAM/throughput cho toàn bộ vector:

```text
SHARDING (chia dữ liệu): chia N vector thành nhiều shard trên nhiều node.
  Query -> gửi tới TẤT CẢ shard (scatter) -> mỗi shard trả top-k cục bộ -> gộp & xếp lại (gather).
  -> scale được N (số vector) và tăng throughput. Latency = shard chậm nhất (fan-out).

REPLICATION (nhân bản): mỗi shard có nhiều bản sao.
  -> HA (1 replica chết vẫn phục vụ) + tăng QPS đọc (chia tải query qua replica).

=> Production: kết hợp cả hai. Ví dụ 4 shard × 2 replica = 8 node.
```

### Cân nhắc sharding
```text
- Số shard: quá ít -> mỗi shard quá to (RAM/latency). Quá nhiều -> overhead fan-out + gather.
- Fan-out latency: query phải chờ shard CHẬM NHẤT -> tail latency (p99) là vấn đề ở nhiều shard.
- Routing: shard theo hash (đều) hoặc theo metadata (vd theo tenant -> query 1 tenant chỉ hit 1 shard).
- Consistency khi ghi: vector mới index vào shard nào, có ngay không (near-real-time vs batch).
```
**Multi-tenancy (hay gặp trong SaaS AI):** thay vì 1 shard chung, cân nhắc **partition theo tenant** (namespace/collection riêng) → query 1 tenant không quét data tenant khác (bảo mật + hiệu năng). Nhưng nhiều tenant nhỏ → nhiều index nhỏ (overhead). Cân bằng theo phân bố tenant.

---

## 4. Hybrid search & Filtering

### Hybrid search (vector + keyword) — tăng chất lượng RAG
```text
Vector search: giỏi ngữ nghĩa (semantic), kém với từ khoá chính xác/hiếm (tên riêng, mã sản phẩm, số).
Keyword (BM25): giỏi khớp chính xác, kém ngữ nghĩa.
Hybrid = kết hợp cả hai -> tốt hơn hẳn cho nhiều truy vấn thực tế.
Gộp điểm: RRF (Reciprocal Rank Fusion) hoặc weighted score.
Thường thêm bước RERANK (cross-encoder) trên kết quả gộp -> tăng độ chính xác top-k.
```
Luồng chất lượng cao: `query → (vector search + BM25) → gộp RRF → rerank cross-encoder → top-k → LLM`.

### Metadata filtering (lọc theo điều kiện) — điểm bẫy hiệu năng
```text
Nhu cầu: "tìm chunk gần nhất NHƯNG chỉ trong tài liệu của tenant X, loại 'policy', sau 2024".
Vấn đề: filter + ANN không đơn giản kết hợp.
  Pre-filter  : lọc trước rồi ANN trên tập con -> đúng nhưng nếu tập con nhỏ, ANN index kém hiệu quả.
  Post-filter : ANN trước rồi lọc -> nhanh nhưng có thể không đủ k sau lọc (phải lấy dư).
  -> Vector DB tốt (Qdrant/Weaviate/Milvus) có filtered-ANN tối ưu sẵn. pgvector kết hợp WHERE + index
     cần cân nhắc kỹ (có thể mất hiệu quả index).
Thiết kế: index metadata (payload) đúng cách; test recall KHI CÓ filter (thường tụt so với không filter).
```

---

## 5. Latency ↔ Recall ↔ Cost (tam giác đánh đổi)

```text
Muốn LATENCY thấp   : giảm ef_search/nprobe, quantize, ít shard fan-out -> recall giảm.
Muốn RECALL cao     : tăng ef_search, dùng vector gốc, rerank -> latency & cost tăng.
Muốn COST thấp (RAM): quantize, IVF-PQ, ít replica -> recall/latency đánh đổi.

KHÔNG có cấu hình "tốt nhất" -> đặt MỤC TIÊU trước:
  vd "p99 < 50ms VÀ recall@10 >= 0.95 với ngân sách X" -> tune tới khi đạt.
```

**Đo lường bắt buộc (không đoán):**
```text
- Recall: tạo ground-truth bằng brute-force trên tập mẫu -> so ANN -> tính recall@k thật.
- Latency: đo p50/p95/p99 dưới tải thật (không chỉ 1 query). Tail latency mới là vấn đề.
- Throughput (QPS): load test -> biết ngưỡng, khi nào cần thêm replica.
- RAM/node: theo dõi để biết khi nào phải shard/quantize.
```

**Tối ưu latency thực chiến:**
```text
- ef_search vừa đủ đạt recall mục tiêu (đừng để cao thừa).
- Two-stage: index nén lọc thô nhanh -> rerank vector gốc (cân bằng tốc độ + recall).
- Giảm số dimension embedding nếu model cho phép (Matryoshka embeddings) -> nhẹ & nhanh hơn.
- Cache kết quả query phổ biến / semantic cache.
- Đặt vector DB gần app (cùng region/AZ) -> giảm network latency.
- Warm index trong RAM (tránh đọc disk cho HNSW).
```

---

## 6. Kiến trúc & vận hành ở quy mô

```text
Ingestion pipeline (offline/near-real-time):
  Source docs -> chunk -> embed (batch, GPU nếu model nặng) -> upsert vào vector DB
             -> versioning knowledge base (rebuild được khi đổi embedding model)

Serving:
  App -> (hybrid: vector DB + BM25) -> rerank -> LLM
  Vector DB: shard + replica trên K8s (StatefulSet + PVC), autoscale replica đọc theo QPS.

Vận hành (Platform Engineer):
  - Re-embedding: đổi embedding model -> PHẢI re-embed toàn bộ (index cũ không tương thích) ->
    kế hoạch reindex không downtime (build index mới song song -> swap).
  - Backup/snapshot collection; disaster recovery.
  - Monitor: latency p99, recall (định kỳ đo lại), QPS, RAM/node, kích thước index.
  - Consistency: độ trễ giữa "ghi vector" và "query thấy được" (near-real-time).
```
> **Bẫy lớn nhất khi vận hành:** đổi embedding model = **re-embed & rebuild toàn bộ index** (vector cũ & mới không cùng không gian). Lên kế hoạch reindex như một migration thật.

---

## 7. Chọn vector DB

| Lựa chọn | Khi nào |
|----------|---------|
| **pgvector** (Postgres) | quy mô vừa (tới ~vài triệu-chục triệu), đã có Postgres, muốn đơn giản, transaction + filter SQL |
| **Qdrant** | chuyên dụng, filtered-ANN tốt, Rust nhanh, self-host/cloud dễ |
| **Weaviate** | có sẵn hybrid + module, GraphQL |
| **Milvus** | quy mô rất lớn (tỷ vector), phân tán mạnh, nhiều index type |
| **Elasticsearch/OpenSearch** | đã dùng cho search/log, cần hybrid vector+keyword trong 1 hệ |
| **Pinecone** | managed, không muốn tự vận hành, trả tiền theo scale |

```text
Nguyên tắc: bắt đầu ĐƠN GIẢN (pgvector nếu đã có Postgres). Chỉ chuyển sang chuyên dụng khi
chạm giới hạn thật (scale, filtered-ANN, hybrid, QPS). Đừng "over-engineer" từ đầu.
```

---

## 8. Checklist vector search ở quy mô
```text
[ ] Đã đo RECALL thật (so brute-force), không tin mặc định.
[ ] ef_search/nprobe tune đạt mục tiêu recall với latency chấp nhận.
[ ] Latency p99 đo dưới tải thật; throughput biết ngưỡng.
[ ] Quantization/IVF chỉ dùng khi RAM/cost thành vấn đề (không sớm).
[ ] Sharding khi 1 node không đủ; replica cho HA + QPS; để ý tail latency fan-out.
[ ] Hybrid (vector + BM25) + rerank nếu chất lượng RAG cần cao.
[ ] Filtered-ANN test recall KHI CÓ filter (thường tụt).
[ ] Kế hoạch RE-EMBED/reindex không downtime khi đổi embedding model.
[ ] Backup/snapshot + monitor (latency/recall/QPS/RAM).
[ ] Vector DB gần app (region/AZ); index warm trong RAM.
```

## Tóm tắt
- Vector search = **ANN** (gần đúng), mọi tuning là đánh đổi **Recall ↔ Latency ↔ Cost**.
- **HNSW** mặc định: nhớ 3 núm M / ef_construction (build) / **ef_search** (runtime — chỉnh trước).
- RAM là ràng buộc ở quy mô → **IVF/quantization + rerank**; nhưng đừng dùng sớm.
- Scale ngang bằng **shard (chia data) + replica (HA/QPS)**; cẩn thận tail latency fan-out.
- **Hybrid + rerank** tăng chất lượng RAG; filtered-ANN phải test recall riêng.
- Bẫy vận hành lớn nhất: đổi embedding model → **re-embed toàn bộ** (migration thật).
