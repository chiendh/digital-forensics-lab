# 08 — MVP Roadmap & Backlog

## 1. Mục tiêu

Chuyển bộ PRD thành **kế hoạch triển khai theo giai đoạn** cho sản phẩm Digital Disk Forensics (Windows), phục vụ cơ quan điều tra / giám định tư pháp.

Mobile Forensic: **không** nằm trong các epic dưới đây (bước tài liệu sau).

---

## 2. Nguyên tắc ưu tiên

1. Custody + hash trước tính năng “đẹp”.
2. Đọc image có sẵn trước khi tối ưu imaging hiện trường.
3. NTFS + exFAT trước filesystem hiếm.
4. FS deleted recovery trước carving toàn đĩa.
5. Report official có provenance trước AI/STIX.

---

## 3. Giai đoạn A — MVP (Workbench giám định tối thiểu)

**Mục tiêu đầu ra:** Giám định viên hoàn thành một case USB/NTFS mẫu: image (hoặc import) → duyệt/phục hồi → finding → PDF official.

### Epic A1 — Case & Custody shell

- Tạo/mở case folder + SQLite
- User profile local + audit_events P0
- UI khung: Case / Evidence / Jobs

**Acceptance:** tạo case, thấy audit `case_created`.

### Epic A2 — Image ingest & verify

- Import DD (split optional sau)
- SHA-256 compute/verify
- Đăng ký `image_assets`

**Acceptance:** import fixture DD; verify pass/fail đúng.

### Epic A3 — Acquisition USB/removable + internal disk

- Device discovery Windows
- Write-blocker acknowledgement
- Stream to DD + bad sector pad_zero + log
- Progress UI

**Acceptance:** image USB mẫu; hash độc lập khớp; custody đủ field.

### Epic A4 — Partitions & NTFS browser

- MBR/GPT list
- NTFS tree allocated
- Preview text/hex/image nhỏ
- Extract allocated + provenance

**Acceptance:** duyệt và extract file biết trước từ image lab.

### Epic A5 — Deleted FS recovery

- NTFS deleted via $MFT (best-effort)
- exFAT/FAT32 deleted entries cơ bản
- Recoverability badges

**Acceptance:** fixture xóa có kiểm soát → liệt kê + extract khi chưa ghi đè.

### Epic A6 — Carving P0 pack

- Unallocated scan: JPEG/PNG/PDF/ZIP-OOXML
- Confidence + extract

**Acceptance:** tìm thấy object carved trên fixture format nhanh.

### Epic A7 — Search / Tag / Finding

- Metadata search
- Tag + Finding confirmed/draft

**Acceptance:** tìm theo tên; tạo finding gắn extract.

### Epic A8 — Timeline nhẹ + Report

- Timeline từ FS timestamps + acquisition
- PDF draft/official + JSON manifest
- Official checklist (hash, findings, custody)

**Acceptance:** xuất official PDF đủ appendix A/B.

### Epic A9 — E01 support (nếu chưa xong trong A2/A3)

- Đọc E01; ghi E01 nếu resource cho phép
- Nếu thiếu thời gian: **đọc E01** là bắt buộc hơn **ghi E01**

---

## 4. Giai đoạn B — Depth forensics disk

| Epic | Nội dung |
|------|----------|
| B1 | USN / $LogFile ingest → timeline |
| B2 | Windows artifact packs: Prefetch, LNK, JumpList, Recycle Bin sâu |
| B3 | Registry + EVTX viewers (nhẹ) |
| B4 | Carving pack mở rộng + custom rules |
| B5 | Pause/resume imaging, split raw, hardware blocker catalog |
| B6 | Supervisor role + hash-chain verify tool |
| B7 | AFF read; BitLocker detect UX cải thiện |
| B8 | STIX/CFO export subset |

---

## 5. Giai đoạn C — Platform

| Epic | Nội dung |
|------|----------|
| C1 | Plugin SDK parser |
| C2 | Multi-user lab server mode (optional) |
| C3 | ext4/APFS (nếu thị trường cần) |
| C4 | Performance: image đa TB, index incremental |
| C5 | Training mode + bundled fixtures từ repo labs |

---

## 6. Giai đoạn D — Liên kết Mobile (sau)

Không thiết kế chi tiết ở đây. Chỉ giữ điểm nối:

- Cùng Case model / custody / report engine
- Mobile acquisition & app parsers là pack riêng
- Tham chiếu `docs/Product_KnowHow_Digital_Mobile_Forensics.md`

---

## 7. Backlog dạng user story (mẫu P0)

1. **Là** giám định viên, **tôi muốn** import file DD vào case và thấy SHA-256, **để** chứng minh tính toàn vẹn.
2. **Là** kỹ thuật viên, **tôi muốn** image USB qua checklist write-blocker, **để** tránh ghi đè nguồn.
3. **Là** giám định viên, **tôi muốn** xem cây NTFS và extract file, **để** đưa vào hồ sơ.
4. **Là** giám định viên, **tôi muốn** liệt kê file đã xóa còn recover được, **để** không bỏ sót evidence.
5. **Là** giám định viên, **tôi muốn** carve JPEG/PDF từ vùng trống, **để** phục hồi khi format nhanh.
6. **Là** giám định viên, **tôi muốn** gắn finding và xuất PDF official, **để** nộp kết quả giám định.

---

## 8. Định nghĩa Done cho MVP (Definition of Done)

- [ ] Chạy trên Windows 10/11 lab chuẩn
- [ ] Case folder tự chứa DB + exports + reports
- [ ] Hỗ trợ DD import + USB imaging + internal disk imaging
- [ ] NTFS + exFAT browse/extract
- [ ] Deleted recovery best-effort có nhãn tin cậy
- [ ] Carving P0 trên unallocated
- [ ] Search metadata + tag + finding
- [ ] Timeline nhẹ
- [ ] PDF official + JSON manifest
- [ ] Audit events P0 đầy đủ
- [ ] Ít nhất 3 fixture kiểm thử tự động (hash, ntfs extract, carve jpeg)
- [ ] Tài liệu người dùng ngắn (1 file) — có thể thêm sau PRD

---

## 9. Rủi ro dự án

| Rủi ro | Mức | Ứng phó |
|--------|-----|---------|
| NTFS edge cases tốn thời gian | Cao | Bọc TSK sớm; giới hạn cam kết |
| E01 write phức tạp | Trung bình | Ưu tiên read; write sau |
| Imaging internal NVMe quyền admin | Trung bình | Document yêu cầu quyền; test matrix |
| False positive carving làm xấu báo cáo | Trung bình | Confidence gate + confirm findings |
| License thư viện mở | Cao | Review pháp lý trước commercial build |
| Scope creep mobile | Cao | Giữ mobile ở giai đoạn D / docs riêng |

---

## 10. Thứ tự đọc khi triển khai sprint 1

1. [01_Product_Overview](01_Product_Overview.md) — đồng bộ phạm vi
2. [07_Data_Model_and_Architecture](07_Data_Model_and_Architecture.md) — tạo skeleton solution
3. [02](02_Acquisition_and_Imaging.md) + [06](06_Custody_Audit_and_Reporting.md) — A1–A3
4. [03](03_Filesystem_and_Deleted_Recovery.md) — A4–A5
5. [04](04_File_Carving_and_Signatures.md) — A6
6. [05](05_Search_Timeline_and_Analysis.md) — A7–A8

---

## 11. Checkpoint sau khi xong bộ tài liệu này

- [ ] Product owner duyệt DoD MVP
- [ ] Chọn UI stack Windows cụ thể
- [ ] Chọn bọc TSK vs tự viết FS (khuyến nghị: bọc)
- [ ] Tạo implementation plan / repo code riêng (không bắt buộc nằm trong repo lab)
