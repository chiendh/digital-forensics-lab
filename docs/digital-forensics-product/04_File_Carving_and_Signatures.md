# 04 — File Carving & Signatures

## 1. Mục tiêu module

Phục hồi file **không còn (hoặc không tin cậy) metadata filesystem**, bằng cách quét byte-stream tìm:

- Header / magic numbers
- Footer / trailer
- Cấu trúc nội bộ (khi cần xác thực)

Áp dụng trên: toàn image, một partition, hoặc **chỉ vùng unallocated** (khuyến nghị mặc định).

---

## 2. Khi nào dùng carving

| Tình huống | FS recovery | Carving |
|------------|-------------|---------|
| File xóa, MFT/FAT còn, cluster chưareuse | Ưu tiên | Bổ sung |
| Format nhanh / xóa bảng FAT | Yếu | Ưu tiên |
| FS hỏng nặng | Không đủ | Ưu tiên |
| File bị đổi tên/extension | FS vẫn đủ | Signature verify hữu ích |
| Ảnh nhúng trong container phức tạp | Tùy | Carving có thể false positive |

**Nguyên tắc UX:** luôn ưu tiên kết quả FS có provenance mạnh; carving gắn nhãn `confidence` và `method=carved`.

---

## 3. Pipeline carving

```text
Chọn phạm vi (unallocated | partition | whole image)
  → Optional: lấy bitmap unallocated từ FS parser
  → Stream scan theo block
  → Signature match (header)
  → Boundary resolve (footer | max-size | parser structure)
  → Validate (magic + light parse)
  → Dedup (hash overlap với file FS đã có)
  → CarvedObject + optional ExtractedFile
```

---

## 4. Signature pack (MVP)

### 4.1. Nhóm P0

| Loại | Magic / gợi ý | Ghi chú |
|------|---------------|---------|
| JPEG | `FF D8 FF` … EOI `FF D9` | Cẩn thận EXIF/thumbnail false split |
| PNG | `89 50 4E 47 0D 0A 1A 0A` … IEND | |
| GIF | `GIF87a` / `GIF89a` | |
| PDF | `%PDF` … `%%EOF` | Có thể nhiều EOF; cần heuristic |
| ZIP / OOXML / APK-like | `PK\x03\x04` | Office xlsx/docx là ZIP |
| OLE Compound (old Office) | `D0 CF 11 E0` | doc/xls/ppt cũ |
| MP4 / ISO BMFF | `ftyp` | Cần đọc box size |
| SQLite | `SQLite format 3` | Hữu ích cross-over mobile/IoT sau này |

### 4.2. P1

- Video khác (AVI, MKV), audio (MP3 frames — dễ false positive), HTML/XML ASCII heuristics, email PST/mbox (phức tạp).

### 4.3. Quản lý signature

- Pack versioned (`signatures_v1.json`).
- Mỗi rule: `id`, `name`, `ext`, `header`, `footer?`, `max_size`, `validator`, `enabled`.
- Lab có thể bật/tắt rule để giảm nhiễu.

---

## 5. Thuật toán biên (boundary)

Theo thứ tự ưu tiên:

1. **Footer match** trong cửa sổ hợp lý.
2. **Cấu trúc tự mô tả** (PNG chunks, MP4 boxes, ZIP central directory).
3. **Max size cut** + đánh `truncated=true`.
4. **Next-header stop** (dừng trước magic của file kế).

Fragmentation: MVP **không** ghép mảnh phức tạp (file carver basic). Ghi rõ giới hạn: file phân mảnh nặng có thể không phục hồi đủ bằng carving tuyến tính.

Tham chiếu ngành: foremost, photorec, binwalk — học hành vi, không cần tái tạo toàn bộ ngay.

---

## 6. Confidence & validation

| Mức | Ý nghĩa | Ví dụ |
|-----|---------|-------|
| `high` | Magic + parser OK + kích thước hợp lý | PNG decode header OK |
| `medium` | Magic + footer hoặc size heuristic | JPEG có EOI |
| `low` | Chỉ header / cắt max-size | PDF không tìm thấy EOF |
| `reject` | Validator fail | “JPEG” không parse được SOF |

Chỉ extract tự động mặc định với `high|medium`; `low` hiện trong danh sách để analyst chọn.

---

## 7. Deduplication

Sau carving:

- Hash nội dung carved.
- So với hash file allocated đã index → đánh dấu `duplicate_of_fs_entry`.
- Tránh làm báo cáo phình bởi cùng một ảnh hai lần trừ khi analyst cố ý.

---

## 8. UX

- Wizard: phạm vi → chọn signature pack → chạy → bảng kết quả (type, offset, size, confidence).
- Preview pane: ảnh/PDF nếu validate OK.
- “Extract selected” / “Extract all high+medium”.
- Link “Show in hex” tại offset.

---

## 9. Yêu cầu chức năng

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| CV-01 | Scan unallocated với pack P0 | P0 |
| CV-02 | Hiển thị offset, length, confidence | P0 |
| CV-03 | Extract + hash + provenance offset | P0 |
| CV-04 | Dedup với FS entries | P1 |
| CV-05 | Custom signature rule import | P1 |
| CV-06 | Binwalk-like entropy map UI | P2 |
| CV-07 | Fragment reassembly nâng cao | P2 |

---

## 10. Tham chiếu repo

- `Basic_Computer_Skills_for_Forensics/9_Data_Carving.pptx`
- `Basic_Computer_Skills_for_Forensics/file_carving/`
- NIST Lab 12 Data Carving
- Tools: `foremost`, `binwalk`

---

## 11. Tiêu chí chấp nhận

- Trên image fixture có file JPEG/PDF đã xóa và vùng MFT/FAT không còn đủ: carving tìm thấy ≥ tỷ lệ đã định trong test plan.
- Mỗi carved export ghi được `byte_offset`, `length`, `rule_id`, `confidence`.
- False positive `low` không bị auto-attach vào báo cáo cuối nếu chưa được analyst duyệt.
