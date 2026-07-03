# Phân biệt các khái niệm ML/AI hay lẫn lộn (cho người đến từ infra/DevOps)

> Người làm hạ tầng thường bị rối vì các thuật ngữ nghe giống nhau: train, fine-tune, training data,
> inference, model, weights... File này phân biệt rõ từng cặp, kèm ví dụ đời thường. Không cần biết toán.

## 0. Bức tranh 1 câu cho mỗi khái niệm

```text
Model         : "bộ não" đã học — một file chứa các con số (weights).
Training      : quá trình DẠY model bằng dữ liệu (điều chỉnh các con số đó).
Training data : dữ liệu DÙNG ĐỂ DẠY.
Inference     : DÙNG model đã học để dự đoán/sinh (chạy thật, phục vụ user).
Fine-tuning   : dạy TIẾP một model đã học sẵn, trên data riêng, để chỉnh hành vi.
Embedding     : biến dữ liệu (text) thành vector số để máy so sánh ngữ nghĩa.
```

---

## 1. Train vs Inference — cặp quan trọng nhất

```text
TRAINING (huấn luyện):
  Đầu vào: RẤT NHIỀU dữ liệu có sẵn -> model ĐIỀU CHỈNH trọng số để "học".
  Đặc điểm: chạy MỘT LẦN (hoặc định kỳ), tốn GPU KHỦNG, kéo dài giờ-ngày-tuần.
  Kết quả: ra một MODEL (file weights).

INFERENCE (suy luận / phục vụ):
  Đầu vào: 1 request của user -> model đã học ĐƯA RA kết quả (không học thêm).
  Đặc điểm: chạy LIÊN TỤC 24/7, cần latency thấp, đây là cái phục vụ production.
  Kết quả: 1 dự đoán/câu trả lời.
```
**Ví dụ đời thường:** Training = **học lái xe** (mất nhiều buổi, vất vả). Inference = **lái xe đi làm** (dùng kỹ năng đã học, mỗi ngày).

**Ý nghĩa hạ tầng (quan trọng với bạn):**
```text
- GPU cho TRAINING: cần nhiều, mạnh, thời gian ngắn (dùng Spot, batch job, xong tắt).
- GPU cho INFERENCE: cần ít hơn/lần nhưng chạy 24/7, tối ưu latency & cost (autoscale, scale-to-zero).
- Đây là 2 loại workload hạ tầng KHÁC NHAU -> file serving ([ml-serving-k8s.md]) lo inference,
  file training ([fine-tuning-lora.md]) lo training.
```

---

## 2. Model vs Weights vs Checkpoint vs Parameters

```text
Model      : nói chung là "cái đã học" — thường gồm kiến trúc (cấu trúc mạng) + weights.
Weights    : các con số bên trong model (kết quả của training). Đây là "kiến thức" của model.
             Model 7B = 7 tỷ con số (parameters). File weights có thể vài GB-vài trăm GB.
Parameters : = weights (số lượng tham số. "7B" = 7 billion parameters -> chỉ độ lớn model).
Checkpoint : bản LƯU của model tại một thời điểm training (để resume hoặc dùng). 
             Training dài -> lưu checkpoint định kỳ (phòng crash, chọn bản tốt nhất).
Artifact   : nói chung mọi thứ sinh ra (model, checkpoint, tokenizer, config).
```
**Ví dụ:** "model Llama 3 8B" = kiến trúc Llama + 8 tỷ weights. File `.safetensors` chứa weights đó.

⚠️ **Lưu ý bảo mật (MLSecOps):** weights lưu dạng `.pkl`/pickle có thể chứa **code độc** khi load → ưu tiên định dạng **safetensors** (chỉ chứa số, không chạy code).

---

## 3. Parameters vs Hyperparameters — dễ nhầm

```text
Parameters (tham số)      : model TỰ HỌC trong lúc training (weights). Bạn không đặt tay.
Hyperparameters (siêu ...) : bạn ĐẶT TAY TRƯỚC khi training, điều khiển CÁCH học.
   ví dụ: learning rate (học nhanh/chậm), batch size, số epoch, rank r của LoRA...
```
**Ví dụ:** Hyperparameter = **cách bạn ôn thi** (học mấy tiếng/ngày, phương pháp). Parameters = **kiến thức trong đầu** sau khi ôn. Bạn chỉnh cách ôn (hyperparameter), kết quả là kiến thức (parameters).

---

## 4. Các loại "training" — từ pre-training tới fine-tuning

```text
1. PRE-TRAINING (huấn luyện gốc):
   Train model TỪ CON SỐ 0 trên hàng nghìn tỷ token (cả internet).
   -> Rất đắt (hàng triệu USD), chỉ big lab (OpenAI/Meta/Google) làm.
   -> Ra "foundation model" (Llama, GPT base). Bạn gần như KHÔNG bao giờ làm cái này.

2. CONTINUED PRE-TRAINING:
   Học tiếp trên corpus domain lớn (vd toàn bộ văn bản y tế) -> vẫn tốn kém, ít gặp.

3. FINE-TUNING (SFT - Supervised Fine-Tuning):
   Lấy foundation model + dạy tiếp trên data RIÊNG (cặp input->output mong muốn).
   -> Chỉnh HÀNH VI/phong cách/format. Đây là cái team thường làm (qua LoRA/QLoRA).

4. PREFERENCE TUNING (RLHF / DPO):
   Căn chỉnh theo sở thích con người (câu trả lời nào "được ưa hơn"). Nâng cao.
```
**Ví dụ:** Pre-training = **học phổ thông 12 năm** (kiến thức nền rộng). Fine-tuning = **học nghề chuyên môn** (lấy nền đó, đào tạo cho công việc cụ thể).

**Nhắc lại điểm cực quan trọng** (xem [fine-tuning-lora.md]):
```text
Fine-tuning KHÔNG hiệu quả để nhồi KIẾN THỨC mới -> dùng RAG cho kiến thức.
Fine-tuning để dạy HÀNH VI/FORMAT/KỸ NĂNG.
```

---

## 5. 3 loại "data" hay bị gộp làm một

```text
TRAINING DATA (pre-training):    khối văn bản khổng lồ để train foundation model (không phải việc bạn).

FINE-TUNING DATA (SFT data):     cặp (input -> output mong muốn) chất lượng cao, VÀI TRĂM-VÀI NGHÌN.
   ví dụ: {"user": "Khách hỏi X", "assistant": "Cách trả lời chuẩn của công ty"}.
   -> dạy model CÁCH cư xử. Cần versioning (DVC).

RAG DATA (knowledge base):       tài liệu/kiến thức để LLM TRA CỨU lúc trả lời (KHÔNG train).
   ví dụ: tài liệu sản phẩm, chính sách, FAQ -> chunk -> embed -> vector DB.
   -> cung cấp KIẾN THỨC lúc inference, cập nhật dễ (chỉ cần thêm/sửa tài liệu, không train lại).
```
| | Fine-tuning data | RAG data |
|--|------------------|----------|
| Mục đích | dạy **hành vi** | cung cấp **kiến thức** |
| Dùng khi nào | lúc **training** | lúc **inference** (query) |
| Cập nhật | phải **train lại** | chỉ **thêm/sửa tài liệu** (dễ) |
| Định dạng | cặp input→output | tài liệu → chunk → vector |

**Đây là chỗ nhầm phổ biến nhất:** "Tôi muốn AI biết tài liệu công ty" → **RAG**, không phải fine-tune.

---

## 6. Các khái niệm training hay gặp (epoch, batch, step, loss)

```text
Epoch : 1 lần model đi qua TOÀN BỘ training data. Train thường vài epoch.
Batch : 1 nhóm mẫu xử lý cùng lúc (vì không nạp hết data 1 lần được). Batch size = số mẫu/lần.
Step  : 1 lần cập nhật weights (sau mỗi batch).
Loss  : "độ sai" của model — training cố GIẢM loss. Loss giảm = model đang học.
        Nhưng loss thấp KHÔNG chắc tốt cho task thật -> phải eval trên validation (xem mục 7).
Learning rate: bước điều chỉnh weights mỗi step (cao -> học nhanh nhưng dễ "văng"; thấp -> chậm, ổn).
Gradient: hướng điều chỉnh weights để giảm loss (toán bên trong, bạn không cần tính tay).
```
**Ví dụ:** Epoch = **đọc hết quyển sách 1 lượt**. Đọc 3 epoch = đọc cả quyển 3 lần. Batch = **đọc từng chương** (không đọc cả sách 1 hơi).

---

## 7. Train / Validation / Test — 3 tập dữ liệu

```text
Training set   : data để DẠY model (model thấy & học từ đây).
Validation set : data để KIỂM TRA TRONG lúc train (chỉnh hyperparameter, phát hiện overfitting).
Test set       : data GIẤU HẲN, chỉ dùng cuối để đánh giá công bằng (model chưa từng thấy).
```
**Vì sao tách:** nếu đánh giá trên chính data đã học → model "học thuộc" mà không giỏi thật (như chấm thi bằng đúng đề đã cho ôn). Test set = đề thi thật chưa từng thấy.

**Overfitting vs Underfitting:**
```text
Overfitting  : học THUỘC LÒNG training data -> giỏi trên data cũ, DỞ trên data mới. (học vẹt)
Underfitting : học chưa đủ -> dở cả trên data cũ lẫn mới. (chưa học tới)
Dấu hiệu overfitting: loss training thấp nhưng loss validation cao/tăng.
```

---

## 8. Embedding vs Token — 2 khái niệm nền của LLM

```text
TOKEN : đơn vị văn bản LLM xử lý (~ một mẩu từ). "Xin chào" có thể = 2-3 token.
        -> LLM tính TIỀN & giới hạn độ dài theo token. "context 8k" = 8000 token.
EMBEDDING : biến text thành VECTOR số (vd 1024 chiều) mã hoá NGỮ NGHĨA.
        -> 2 câu nghĩa giống nhau -> 2 vector gần nhau -> đó là nền của semantic search/RAG.
```
**Khác nhau:** Token = cách LLM **đọc/đếm** văn bản. Embedding = cách biến văn bản thành **số để so sánh ý nghĩa**. Cả hai đều "biến text thành số" nhưng mục đích khác: token để model xử lý tuần tự, embedding để đo độ giống.

---

## 9. Bảng tra nhanh "khi nào dùng gì" (tổng kết cho DevOps AI)

| Bạn muốn... | Dùng | KHÔNG dùng |
|-------------|------|------------|
| AI biết tài liệu/kiến thức riêng, cập nhật thường | **RAG** | fine-tune |
| AI trả lời đúng format/giọng điệu ổn định | prompt → **fine-tune** nếu cần | RAG |
| Chạy model phục vụ user 24/7 | **inference/serving** | training |
| Dạy model kỹ năng mới từ ví dụ | **fine-tuning (LoRA)** | RAG |
| So sánh độ giống ngữ nghĩa của text | **embedding + vector search** | |
| Model nền thông minh sẵn | **foundation model** (dùng qua API/self-host) | tự pre-train |

## Tóm tắt (nhớ 4 cặp then chốt)
```text
1. Training (dạy, tốn GPU, 1 lần) vs Inference (dùng, 24/7, phục vụ user).
2. Fine-tuning (dạy HÀNH VI) vs RAG (cung cấp KIẾN THỨC).  <- nhầm nhiều nhất
3. Parameters (model tự học) vs Hyperparameters (bạn đặt tay).
4. Fine-tuning data (input→output, lúc train) vs RAG data (tài liệu, lúc query).
```
