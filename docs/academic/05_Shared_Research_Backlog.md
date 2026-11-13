# 05 — Shared Research Backlog

Việc nghiên cứu / tài liệu còn lại sau khi đã có: Disk PRD, Mobile PRD, academic charter, eval, STIX/PreDIE.

---

## P0 — Cần cho đánh giá & phát triển gần

| ID | Hạng mục | Ghi chú |
|----|----------|---------|
| R-01 | `expected_findings` cho F-DISK-USB và F-AND-10 | JSON gold tối thiểu |
| R-02 | Catalog path/DB Android tường minh | Bổ sung từ PPT/corpora: mmssms, contacts2, … |
| R-03 | Catalog path/DB iOS tường minh | sms.db, knowledgeC.db, … |
| R-04 | Bảng coverage DFAG v0 | Markdown trong docs/academic |
| R-05 | License review TSK/libewf trước commercial | Ngoài repo lab nhưng blocker App1 |

## P1 — Làm giàu học thuật

| ID | Hạng mục |
|----|----------|
| R-10 | Rút artifact tables từ PPT Android/iOS → CSV (script bán tự động nếu có OCR) |
| R-11 | Echo lab_data → inventory máy đọc được (file list + hash) |
| R-12 | Map STIX JSON Rhino → schema Case DB App1 (field-level) |
| R-13 | Ablation note zero/one-shot từ AI4Forensics notebooks |
| R-14 | Gap matrix iOS vs Cellebrite/AXIOM (điền sau khi chạy corpora) |

## P2 — Mở rộng

| ID | Hạng mục |
|----|----------|
| R-20 | Multi-device graph (group crime / CTF p2) |
| R-21 | Drone GPS↔image correlation methodology note |
| R-22 | Memory forensics track gắn App1 giai đoạn C |
| R-23 | Network PCAP add-on spec |
| R-24 | Vietnamese forensic report language pack |

---

## Việc **không** làm trong repo lab này

- Commit binary image lớn (tuân thủ nguồn Digital Corpora / link)  
- Re-implement toàn bộ decoder WhatsApp thương mại không có quyền  
- Tự động hóa chip-off phần cứng  

---

## Trạng thái hoàn thành tài liệu (checklist)

- [x] Disk product PRD 00–08  
- [x] Academic charter + mapping + eval + STIX  
- [x] Mobile product PRD 00–08  
- [x] Artifact catalog starter (06) — Maps/Echo confirmed; Android/iOS path verify_on_corpora  
- [ ] Gold fixtures expected_findings (R-01…)  
- [ ] Artifact path catalogs điền đủ từ corpora (R-02, R-03)  
