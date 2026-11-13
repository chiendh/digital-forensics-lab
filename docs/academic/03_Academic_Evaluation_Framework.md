# 03 — Academic Evaluation Framework

Khung đánh giá dùng cho: **bài lab sinh viên**, **paper/tech report**, và **regression hai ứng dụng** phục hồi dữ liệu.

---

## 1. Ba lớp đánh giá

| Lớp | Câu hỏi | Đầu ra |
|-----|---------|--------|
| **L1 — Technique** | Parser/carving/hash có đúng trên fixture không? | Metrics P/R/F1, hash match |
| **L2 — Investigation** | Analyst dựng được câu chuyện có nguồn không? | Timeline, findings, report rubric |
| **L3 — Presentability (PreDIE)** | Bằng chứng trình bày được / kiểm chứng được không? | Checklist 5 tiêu chí |

PreDIE chi tiết: [04_Evidence_Knowledge_Graph_and_STIX.md](04_Evidence_Knowledge_Graph_and_STIX.md).

---

## 2. Rubric kỹ thuật (L1) — dùng chung Disk & Mobile

### 2.1. Integrity

| Metric | Cách đo | Pass gợi ý |
|--------|---------|------------|
| Image hash stable | SHA-256 tool app vs hashdeep/Get-FileHash | 100% match |
| Re-verify on open | Mở lại case | status `verified` |
| Export hash logged | Mọi extract có SHA-256 trong manifest | 100% |

### 2.2. Filesystem / logical recovery (Disk + mobile FS dump)

| Metric | Cách đo |
|--------|---------|
| Entry recall | So cây file với `fls`/TSK hoặc gold list |
| Deleted recall | Fixture xóa có kiểm soát |
| Content integrity | Hash file extract == hash ground truth khi chưa overwrite |

### 2.3. Carving

| Metric | Cách đo |
|--------|---------|
| Precision@confidence≥medium | TP / (TP+FP) trên tập gắn nhãn |
| Recall theo loại (JPEG/PDF/…) | TP / (TP+FN) |
| Provenance | Mọi carved object có offset+length+rule_id |

### 2.4. Mobile artifact parsers

| Metric | Cách đo |
|--------|---------|
| Message/thread recall | So số thread/message với gold hoặc AXIOM/Cellebrite export (nếu có quyền) |
| Timestamp normalization | % event parse được → UTC hợp lệ |
| Coordinate decode | Maps: lat/lon sai số < epsilon (vd. 1e-5°) |
| Schema resilience | Parser không crash trên DB version N±1; ghi `unsupported` rõ |

### 2.5. Search

| Metric | Cách đo |
|--------|---------|
| Keyword hit rate | Trên corpus gieo chuỗi biết trước |
| False alarm control | Regex IP/email trên negative set |

---

## 3. Rubric điều tra (L2)

Thang 0–4 cho mỗi tiêu chí (0 thiếu, 4 xuất sắc):

1. **Device/Source dossier** — nhận diện đúng nguồn (disk serial / Android build / iOS image id).  
2. **Artifact inventory** — bảng artifact có path/DB/table.  
3. **Interpretation** — giải thích ý nghĩa forensic (không chỉ dump SQL).  
4. **Timeline** — ≥ N sự kiện đa nguồn, có filter thời gian.  
5. **Method justification** — vì sao logical vs carve vs root/jailbreak/chip-off.  
6. **Tool triangulation** — đối chiếu ≥1 tool độc lập hoặc query thủ công.  
7. **Report quality** — đủ section; không copy raw không chú thích.

**Điểm lab học thuật đề xuất:** trọng số đều hoặc nhấn (3)(4)(7) tùy môn.

---

## 4. Rubric PreDIE (L3) — cổng trước “official report”

| Tiêu chí | Câu hỏi kiểm | Evidence trong app |
|----------|---------------|--------------------|
| **Authenticity** | Finding truy về image/offset/fs_ref? | provenance_json |
| **Integrity** | Hash image/export còn khớp? | verify_status |
| **Validity** | Tool+version nào sinh ra? | engine_versions |
| **Credibility** | Ai confirm finding? | confirmed_by |
| **Relevance** | Có đường dẫn finding → action → case? | finding_evidence + timeline link |

Official export **fail** nếu thiếu Authenticity/Integrity/Validity trên bất kỳ finding confirmed.

---

## 5. Bộ fixture khuyến nghị (tối thiểu)

| ID | Nguồn | App | Mục đích |
|----|-------|-----|----------|
| F-DISK-USB | `Basic_…/dd_image` hoặc image tự tạo | App1 | Imaging+hash+browse |
| F-DISK-CARVE | `file_carving` / NIST lab 12 style | App1 | Carving precision |
| F-AND-10 | Digital Corpora Android 10 (Hickman) | App2 | Native+GMS packs |
| F-IOS-13 | Digital Corpora iOS 13 | App2 | Messages/KnowledgeC |
| F-ECHO-UD | Echo userdata subset `lab_data` | App2 IoT | XML/SQLite/carve |
| F-STIX-RHINO | `STIX_…/illegal_possession_image.json` | Shared | Export shape |

Mỗi fixture cần file `expected_findings.md` hoặc JSON gold (tạo dần trong backlog).

---

## 6. So sánh tool thương mại (gap analysis)

Dùng iOS labs 13–14 làm mẫu quy trình:

| Hạng mục | Thủ công/lab OSS | App đang phát triển | Cellebrite/AXIOM |
|----------|------------------|---------------------|------------------|
| Acquisition breadth | Thấp–TB | ? | Cao |
| App coverage | Theo bài giảng | Parser packs | Rộng |
| Deleted SQLite | Carving/undark thủ công | Mục tiêu P1 | Thường có |
| Report/custody | Template PPT/PDF | Module 06 | Có |
| Court-oriented KG | STIX research | Roadmap | Khác tiếp cận |

**Quy tắc công bố:** chỉ ghi “hỗ trợ artifact X trên image Y phiên bản Z”, không ghi “tương đương Cellebrite”.

---

## 7. Đánh giá AI assist (khi bật)

Từ `AI4Forensics/CKIM2024`:

| Metric | Mô tả |
|--------|-------|
| Entity P/R | So gold conversation/phishing set |
| Edge P/R | Quan hệ đúng/sai |
| Zero-shot vs one-shot | Ablation chất lượng STIX draft |
| Confirmation rate | % suggestion được human accept |
| Hallucination incidents | Finding không có provenance — phải = 0 ở chế độ official |

AI **không** tham gia điểm L1 integrity.

---

## 8. Báo cáo đánh giá mẫu (cho paper / nội bộ)

1. Mục tiêu & phạm vi app  
2. Fixtures & ethics (dataset công khai)  
3. Metrics L1 bảng số  
4. Case walkthrough L2  
5. PreDIE compliance  
6. Gap vs commercial / vs manual TSK  
7. Hạn chế & đe dọa validity (TRIM, encryption, schema drift)  
8. Kết luận phục vụ phát triển tiếp  

---

## 9. Definition of Done — “đủ để đánh giá học thuật một phiên bản engine”

- [ ] ≥ 3 fixture tự động (hash, FS/logical extract, một parser hoặc carve)  
- [ ] Bảng coverage artifact công bố  
- [ ] Một case report mẫu đạt PreDIE  
- [ ] Gap matrix cập nhật  
- [ ] Phiên bản tool/engine ghi trong mọi official export  
