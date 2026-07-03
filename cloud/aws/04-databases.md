# 04 — Databases (RDS, Aurora, DynamoDB, ElastiCache, Analytics)

> Chọn đúng database cho đúng bài toán là kỹ năng lõi của Solutions Architect.
> Nguyên tắc AWS: **"purpose-built databases"** — mỗi loại workload có 1 DB tối ưu riêng.

## 0. Phân loại nhanh
```text
Relational (SQL)     : RDS, Aurora            -> quan hệ, giao dịch ACID, schema cố định
Key-Value (NoSQL)    : DynamoDB               -> quy mô lớn, latency ms, serverless
In-memory            : ElastiCache (Redis/Memcached), MemoryDB -> cache, tốc độ micro giây
Document             : DocumentDB (MongoDB-compatible)
Wide-column          : Keyspaces (Cassandra)
Graph                : Neptune                -> quan hệ mạng, social, fraud
Time-series          : Timestream            -> IoT, metric
Ledger               : QLDB                  -> sổ cái bất biến, có xác thực
Data Warehouse       : Redshift              -> analytics/OLAP quy mô petabyte
Search               : OpenSearch            -> full-text, log analytics
```

## 1. RDS — Relational Database Service (managed SQL)

```text
Managed: MySQL, PostgreSQL, MariaDB, Oracle, SQL Server (và Aurora).
AWS lo: patch OS/engine, backup, failover, monitor. Bạn lo: schema, query, tuning app.
```
### HA & Scaling (điểm thi cực nặng — phân biệt rõ)
| Tính năng | Mục đích | Cơ chế |
|-----------|----------|--------|
| **Multi-AZ** | **HA / chịu lỗi** (KHÔNG phải để đọc) | standby ở AZ khác, **đồng bộ (sync)**, tự failover khi primary chết; standby không phục vụ read |
| **Read Replica** | **Scale đọc** (offload read) | replica **bất đồng bộ (async)**, đọc được, có thể cross-region; **không tự failover** |
- **Bẫy kinh điển:** "cần chịu lỗi/HA" → **Multi-AZ**. "DB đọc quá tải, cần scale read" → **Read Replica**. Đừng nhầm.
- Read Replica có thể **promote** thành DB độc lập (dùng cho migrate/DR thủ công).
- **Backup**: automated backups (point-in-time recovery tới từng giây trong retention) + manual snapshot (giữ tới khi xóa). Snapshot copy cross-region cho DR.
- **Storage**: gp3/io1 autoscaling. **Encryption** KMS (bật lúc tạo; bật sau phải qua snapshot).
- **RDS Proxy**: pool kết nối (giảm tải connection, hợp Lambda), failover nhanh hơn.

## 2. Aurora — RDS "xịn" của AWS
```text
Tương thích MySQL/PostgreSQL nhưng kiến trúc riêng: storage tách khỏi compute,
tự replicate 6 bản qua 3 AZ, tự lành, tự scale storage (tới 128TB).
Nhanh hơn RDS thường (3-5x MySQL), tới 15 read replica, failover < 30s.
```
- **Aurora Serverless v2**: tự co giãn compute theo tải (workload bùng/không đều, dev/test).
- **Global Database**: replicate cross-region, latency thấp, DR (RPO/RTO thấp) — điểm thi SAP.
- **Aurora Replicas** dùng chung storage → failover nhanh & không mất data như async.
- Chọn Aurora khi: cần hiệu năng/độ bền cao hơn RDS, sẵn sàng trả nhiều hơn, vẫn muốn SQL.

## 3. DynamoDB — NoSQL key-value serverless
```text
Fully managed, serverless, latency ms ổn định ở MỌI quy mô, tự replicate nhiều AZ.
Không có server để quản, không patch. Trả theo capacity hoặc request.
```
### Khái niệm cốt lõi
- **Table** → **Item** (row) → **Attributes**. **Primary key**: Partition key (hash) hoặc Partition + Sort key (composite).
- **Partition key phải phân tán tốt** → tránh "hot partition" (điểm thiết kế quan trọng).
- **Capacity mode**: **On-Demand** (tự scale, trả theo request — tải không đoán được) vs **Provisioned** (đặt RCU/WCU + auto scaling — tải ổn định, rẻ hơn).
- **GSI** (Global Secondary Index): query theo attribute khác PK (partition khác). **LSI** (Local): cùng partition, khác sort key.
- **DynamoDB Streams**: bắt thay đổi (insert/update/delete) → trigger Lambda (event-driven).
- **DAX**: cache in-memory cho DynamoDB (đọc micro giây).
- **Global Tables**: multi-region active-active (latency thấp toàn cầu + DR).
- **TTL**: tự xóa item hết hạn (rẻ, dọn data cũ).
- **Transactions**: ACID cho nhiều item.

**Chọn DynamoDB khi:** cần scale khổng lồ, latency ms ổn định, serverless, access pattern rõ (key-value/lookup). **Không hợp:** query phức tạp/ad-hoc join (đó là việc của SQL).

## 4. ElastiCache — cache in-memory
| Engine | Đặc điểm |
|--------|----------|
| **Redis** | giàu tính năng: replication, HA (Multi-AZ + failover), pub/sub, sorted set, persistence, cluster mode |
| **Memcached** | đơn giản, multi-thread, sharding — pure cache, không HA/persistence |
- Dùng để: giảm tải DB (cache kết quả query), session store, leaderboard, rate limiting.
- **Caching patterns:** Lazy loading (cache-aside) vs Write-through. TTL để tránh stale.
- **MemoryDB for Redis**: Redis nhưng **bền** (primary DB được, không chỉ cache).

## 5. Analytics & Data (biết để nhận diện trong đề)
- **Redshift**: data warehouse (OLAP) petabyte, columnar, MPP. **Spectrum** query trực tiếp trên S3. Cho BI/report, không cho OLTP.
- **Athena**: query SQL **serverless trực tiếp trên S3** (trả theo data quét). Kết hợp **Glue Data Catalog**. Ad-hoc, log analytics.
- **Glue**: ETL serverless + Data Catalog (schema).
- **EMR**: Hadoop/Spark managed (big data).
- **Kinesis**: streaming realtime — **Data Streams** (ingest, tự quản shard), **Firehose** (nạp vào S3/Redshift/OpenSearch, không quản), **Data Analytics** (SQL/Flink trên stream).
- **MSK**: Kafka managed.
- **QuickSight**: BI dashboard.
- **Lake Formation**: dựng data lake + phân quyền tập trung trên S3.

## 6. Migration (SAP hay hỏi)
- **DMS** (Database Migration Service): migrate DB (homogeneous & heterogeneous), có thể chạy liên tục (CDC) ít downtime.
- **SCT** (Schema Conversion Tool): chuyển schema giữa engine khác nhau (Oracle → PostgreSQL).

---

## Cây quyết định chọn database
```text
Quan hệ, giao dịch ACID, SQL                 -> RDS (hoặc Aurora nếu cần hiệu năng/độ bền cao)
Cần HA DB SQL                                -> Multi-AZ.  Cần scale đọc -> Read Replica
NoSQL scale lớn, latency ms, serverless      -> DynamoDB
Cache tăng tốc / session                     -> ElastiCache (Redis)
Cache bền làm primary                        -> MemoryDB
Analytics/BI petabyte (OLAP)                 -> Redshift
Query ad-hoc SQL trên S3                     -> Athena
Streaming realtime                           -> Kinesis / MSK
Graph / social / fraud                       -> Neptune
Time-series / IoT                            -> Timestream
Document (Mongo)                             -> DocumentDB
Sổ cái bất biến, audit                       -> QLDB
```

## Tóm tắt điểm bẫy
- **Multi-AZ = HA (sync, không đọc được)** vs **Read Replica = scale đọc (async)**. Nhớ kỹ.
- Aurora: 6 bản/3 AZ, storage tách compute, Global Database cho DR cross-region.
- DynamoDB: thiết kế **partition key phân tán**, On-Demand vs Provisioned, Streams cho event, Global Tables cho multi-region.
- Athena = serverless query trên S3; Redshift = warehouse OLAP.
- Backup: RDS automated (PITR) + snapshot; encryption bật **lúc tạo**.
