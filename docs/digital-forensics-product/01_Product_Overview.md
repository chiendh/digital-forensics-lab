# 01 — Product Overview: Digital Disk Forensics Workbench

## 1. Tóm tắt sản phẩm

**Digital Disk Forensics Workbench** (tên làm việc trong tài liệu) là ứng dụng desktop Windows hỗ trợ giám định viên:

1. **Thu thập (acquire)** dữ liệu từ ổ cứng gắn trong, USB, thẻ nhớ và nhận **disk image** có sẵn.
2. **Bảo toàn tính toàn vẹn** (hash, custody, audit).
3. **Trích xuất / phục hồi** file còn và đã xóa (filesystem metadata + carving).
4. **Phân tích hỗ trợ** (search, tag, timeline nhẹ).
5. **Xuất báo cáo giám định** có provenance đầy đủ.

Sản phẩm **không** thay thế quy trình pháp lý; nó **ghi nhận và hỗ trợ** quy trình đó bằng bằng chứng kỹ thuật có thể kiểm chứng.

---

## 2. Vấn đề cần giải quyết

Giám định viên thường phải:

- Ghép nhiều tool rời (imaging, FTK Imager-class, Autopsy/Sleuth Kit, foremost/binwalk, spreadsheet báo cáo).
- Tự theo dõi hash / nhật ký thao tác — dễ thiếu sót khi áp lực thời gian.
- Phục hồi dữ liệu từ thẻ nhớ/USB bị xóa format hoặc file system hỏng.
- Giải thích nguồn gốc từng file xuất ra trong hồ sơ giám định.

**Giá trị sản phẩm:** một workbench thống nhất từ imaging → recovery → report, với custody là mặc định chứ không phải bước phụ.

---

## 3. Persona & nhu cầu

### 3.1. Giám định viên chính (Primary)

- Làm việc theo lệnh trưng cầu / quyết định tố tụng.
- Cần: write-blocker, hash, log, báo cáo mẫu, khả năng tái lập kết quả.
- Đánh giá thành công: “Tôi bảo vệ được tính toàn vẹn evidence và giải thích được từng finding.”

### 3.2. Trưởng lab / kiểm soát chất lượng

- Cần: audit trail, phiên bản phần mềm/engine trên mọi báo cáo, chuẩn hóa quy trình.
- Đánh giá thành công: “Hai giám định viên cùng image cho cùng hash và cùng danh sách finding cốt lõi.”

### 3.3. Kỹ thuật viên thu thập hiện trường (Secondary)

- Cần: quy trình imaging rõ ràng, cảnh báo ghi đè, bad sector handling.
- Đánh giá thành công: “Tạo được image hợp lệ mà không làm hỏng nguồn.”

---

## 4. Phạm vi

### In scope (bộ tài liệu & MVP hướng tới)

| Hạng mục | Chi tiết |
|----------|----------|
| Nguồn | Ổ cứng gắn trong, USB, thẻ nhớ (SD/microSD qua reader), file image |
| Định dạng image | DD/raw (bắt buộc), E01 (MVP hoặc sớm giai đoạn B), AFF (sau) |
| Filesystem | NTFS (ưu tiên), exFAT/FAT32 (thẻ nhớ/USB) |
| Recovery | Deleted via FS metadata + signature carving |
| Phân tích | Search, tag, timeline từ timestamp FS + sự kiện thủ công |
| Đầu ra | Export file + PDF/HTML report + custody log |
| OS | Windows desktop (MVP) |

### Out of scope (đợt này)

- Mobile Forensic chi tiết (Android/iOS logical/physical parsers)
- Memory forensics sâu (Volatility-class) — có thể giai đoạn sau
- Network PCAP analysis chuyên sâu
- Cloud mailbox / SaaS eDiscovery
- Multi-tenant SaaS; sản phẩm là **desktop / lab-installed**
- Tự động “kết luận tội danh” — chỉ hỗ trợ finding kỹ thuật

---

## 5. Nguyên tắc sản phẩm (non-negotiable)

1. **Never write to evidence source** trừ khi người dùng vượt cảnh báo nguy hiểm có chủ đích (mặc định chặn).
2. **Hash early, verify often** — lúc tạo image, lúc mở case, lúc xuất finding quan trọng.
3. **Every exported byte has provenance** — image id, partition, offset hoặc FS reference, tool version.
4. **Audit is append-only** — không xóa log thao tác.
5. **Reproducibility** — cùng engine version + cùng image → kết quả parse/carve nhất quán trong phạm vi đã công bố.
6. **Human confirms legal narrative** — phần mềm không thay thế nhận định giám định viên.

---

## 6. Luồng sử dụng chính (happy path)

```text
Tạo Case
  → Thêm Evidence Source (disk / removable / existing image)
  → Acquisition (nếu cần) + ghi hash + custody entry
  → Mở Image → liệt kê Partition/Volume
  → Duyệt FS / đánh dấu file quan tâm / phục hồi deleted
  → Chạy Carving trên unallocated (nếu cần)
  → Search + Tag + gắn vào Timeline
  → Export files + sinh Báo cáo giám định
```

---

## 7. Tiêu chí thành công sản phẩm (product success metrics)

| Chỉ số | Mục tiêu định hướng |
|--------|---------------------|
| Thời gian từ cắm USB → có image đã hash | Đủ nhanh cho hiện trường; có progress + ước lượng |
| Tỷ lệ case có đủ custody log bắt buộc | 100% case đóng báo cáo |
| Phủ filesystem MVP | NTFS + exFAT/FAT32 ổn định |
| Phục hồi carving | Ảnh phổ biến + PDF + Office signatures với false-positive có gắn nhãn |
| Báo cáo | Một lệnh xuất đủ: mô tả case, hash, findings, appendix |

---

## 8. Đối sánh với hệ sinh thái (định vị)

Sản phẩm học hỏi quy trình từ lab/tool mở trong repo (Sleuth Kit, foremost/binwalk, hashdeep, ewf-tools…) nhưng định vị là **workbench giám định tích hợp**, không phải bộ sưu tập lệnh CLI.

Tham chiếu học trong repo:

- USB Image Acquisition, Data Carving, Sleuth Kit tutorials
- NIST / P2P leakage cases (disk-scale investigation patterns)
- STIX/CFO (chuẩn hóa evidence objects — giai đoạn B+)

---

## 9. Liên kết tài liệu

- Acquisition chi tiết → [02](02_Acquisition_and_Imaging.md)
- FS & deleted → [03](03_Filesystem_and_Deleted_Recovery.md)
- Carving → [04](04_File_Carving_and_Signatures.md)
- Analysis UX → [05](05_Search_Timeline_and_Analysis.md)
- Custody & report → [06](06_Custody_Audit_and_Reporting.md)
- Architecture → [07](07_Data_Model_and_Architecture.md)
- Roadmap → [08](08_MVP_Roadmap_and_Backlog.md)
