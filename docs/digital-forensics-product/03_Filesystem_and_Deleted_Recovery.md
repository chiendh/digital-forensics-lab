# 03 — Filesystem Analysis & Deleted File Recovery

## 1. Mục tiêu module

Sau khi có `ImageAsset`, module này:

1. Đọc bảng phân vùng (MBR/GPT).
2. Nhận diện và gắn volume (NTFS, exFAT, FAT32…).
3. Dựng **cây thư mục** (allocated).
4. Phát hiện **file/thư mục đã xóa** còn dấu vết metadata.
5. Cho phép **extract** ra storage phân tích kèm provenance.

Đây là lớp phục hồi “có cấu trúc” — khác carving (file không còn metadata đủ tin cậy).

---

## 2. Pipeline kỹ thuật

```text
ImageAsset
  → Partition Table Parser (MBR/GPT)
  → Volume Detector (filesystem type, start LBA, length)
  → FS Parser (NTFS / exFAT / FAT)
  → Entry Index (path, attrs, timestamps, size, deleted flag)
  → Content Reader (data runs / clusters)
  → ExtractedFile (optional export)
```

---

## 3. Partition & volume

### 3.1. Yêu cầu

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| FS-01 | Parse MBR và GPT | P0 |
| FS-02 | Liệt kê partition: index, type, start, size, FS hint | P0 |
| FS-03 | Hỗ trợ volume không trong bảng (heuristic) | P1 |
| FS-04 | Hiển thị bitlocker/encrypted indicator | P1 |

### 3.2. UX tối thiểu

- Sidebar: Image → Partitions → Volume root.
- Badge: `Healthy` / `Damaged FS` / `Unrecognized` / `Encrypted`.

---

## 4. NTFS (ưu tiên P0)

### 4.1. Đối tượng cần đọc

| Cấu trúc | Mục đích |
|----------|----------|
| Boot sector / $Volume | Tham số volume |
| $MFT | Nguồn sự thật cho file records |
| $MFTMirr | Dự phòng khi $MFT hỏng một phần |
| Filename + DATA attributes | Tên, kích thước, data runs |
| SI / FN timestamps | Timeline (lưu ý sự khác nhau SI vs FN) |
| $Bitmap | Cluster allocated vs free (phục vụ carving vùng trống) |
| $I30 / Index | Directory index; có thể còn tên đã xóa |

### 4.2. Deleted recovery trên NTFS

Chiến lược MVP:

1. Quét $MFT tìm record có cờ deleted / không còn trong directory index.
2. Nếu data runs còn và cluster chưa bị ghi đè (best-effort check qua $Bitmap): đánh `recoverable=likely`.
3. Nếu runs thiếu / cluster reused: đánh `recoverable=partial|unlikely`, vẫn giữ metadata.

**Không cam kết 100% nội dung** — UI phải hiện mức tin cậy.

### 4.3. Timestamp

Chuẩn hóa về UTC trong DB. Giữ nguyên giá trị gốc + nguồn (`SI_mtime`, `FN_ctime`, …) để giám định viên giải thích.

Tham chiếu lab: NIST labs về $MFT, USN (USN sâu = giai đoạn B).

---

## 5. exFAT / FAT32 (P0 cho thẻ nhớ & USB)

| Khả năng | FAT32 | exFAT |
|----------|-------|-------|
| Cây thư mục allocated | P0 | P0 |
| Deleted entry (đánh dấu tên) | P0 | P0 |
| Phục hồi cluster chain | P0 khi chain còn | P0 khi còn |
| Fragment phức tạp / hỏng FAT | P1 best-effort | P1 |

Thẻ nhớ máy ảnh thường exFAT — đây là lý do P0 dù NTFS là trọng tâm PC.

---

## 6. FS Entry model (logic)

Mỗi entry trong index:

```text
FsEntry
  id, volume_id
  parent_id / path
  name, is_dir
  is_deleted
  size_reported
  timestamps{}          # map nguồn → UTC
  fs_ref                # MFT record no / DIR entry offset
  data_location         # runs / first cluster
  overwrite_estimate    # unknown|none|partial|full
  recoverability        # allocated|deleted_likely|deleted_unlikely|metadata_only
```

---

## 7. Extract / export file

Khi người dùng extract:

1. Đọc bytes theo data runs (hoặc chain).
2. Ghi ra thư mục case `exports/`.
3. Hash file xuất (SHA-256).
4. Tạo `ExtractedFile` liên kết `FsEntry` + `ImageAsset`.
5. Audit event `file_extracted`.

**Slack space (P1):** tùy chọn extract slack của cluster cuối.

---

## 8. Damaged filesystem modes

| Chế độ | Hành vi |
|--------|---------|
| Normal mount-like parse | Đủ metadata sạch |
| Degraded MFT scan | Bỏ qua index directory; liệt kê từ $MFT thô |
| Unallocated-only | Không FS parser; chuyển sang carving ([04](04_File_Carving_and_Signatures.md)) |

UI luôn cho đường thoát sang carving khi FS không parse được.

---

## 9. Giới hạn MVP (nói rõ với khách hàng nội bộ)

- Chưa: APFS, ext4, HFS+ (roadmap B/C).
- Chưa: dedup / ReFS đầy đủ.
- Chưa: deep $LogFile / USN journal timeline (P1/P2 — xem [05](05_Search_Timeline_and_Analysis.md)).
- Soft-deleted vs secure-wiped: secure wipe → chỉ còn carving yếu hoặc không còn.

---

## 10. Tham chiếu repo

- Sleuth Kit tutorial: `Basic_Computer_Skills_for_Forensics/6_Sleuth_Kit_Tutorial.pptx`
- NIST `$MFT` / Recycle Bin / anti-forensics labs
- Tools: `analyzeMFT`, `INDXParse`, `fls`/`icat` (TSK)

Khuyến nghị kỹ thuật: giai đoạn đầu có thể **bọc The Sleuth Kit / libtsk** hoặc thư viện tương đương thay vì viết NTFS từ zero; vẫn cần lớp model/provenance của sản phẩm.

---

## 11. Tiêu chí chấp nhận

- Mở image NTFS mẫu: duyệt được cây thư mục, mở preview text/ảnh nhỏ.
- Xóa file trong fixture có kiểm soát → sản phẩm liệt kê deleted và extract được nếu chưa ghi đè.
- Volume exFAT trên image thẻ nhớ: liệt kê được file ảnh allocated.
- Mọi file extract có hash + fs_ref trong báo cáo/export manifest.
