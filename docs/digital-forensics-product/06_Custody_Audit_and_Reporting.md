# 06 — Custody, Audit & Reporting

## 1. Mục tiêu module

Đảm bảo mọi thao tác trên evidence **giải trình được** và đầu ra giám định **đủ yếu tố pháp lý kỹ thuật**:

- Chain of custody
- Audit log append-only
- Báo cáo + manifest export
- Phiên bản phần mềm / engine trên mọi hồ sơ chính thức

---

## 2. Chain of custody

### 2.1. Sự kiện custody bắt buộc

| Sự kiện | Khi nào |
|---------|---------|
| `case_created` | Tạo case |
| `evidence_registered` | Thêm nguồn / image |
| `acquisition_started` / `acquisition_completed` / `acquisition_failed` | Imaging |
| `hash_computed` / `hash_verified` / `hash_mismatch` | Hash |
| `blocker_acknowledged` | Xác nhận write-blocker |
| `analysis_opened` | Mở image trong case |
| `file_extracted` | Export nội dung |
| `finding_confirmed` | Finding chuyển confirmed |
| `report_exported` | Xuất báo cáo |
| `case_closed` | Đóng case (optional) |

### 2.2. Trường tối thiểu mỗi custody/audit entry

```text
event_id
ts_utc
actor_user_id
actor_display_name
case_id
event_type
object_refs[]          # image, export, finding…
details_json           # hash, path, reason…
tool_name
tool_version
engine_versions{}      # fs_parser, carver, ...
prev_hash?             # optional hash-chain cho chống sửa log
```

### 2.3. Hash-chain log (khuyến nghị P0/P1)

Mỗi entry chứa `entry_hash = H(prev_hash + canonical_payload)`.  
Không ngăn được admin DB tấn nhưng phát hiện chỉnh sửa thầm.

---

## 3. Vai trò người dùng (MVP đơn giản)

| Vai trò | Quyền |
|---------|--------|
| `examiner` | Làm việc case được giao: acquire, analyze, draft report |
| `supervisor` | Override hash mismatch / training bypass; duyệt report |
| `viewer` | Chỉ đọc (P1) |

MVP có thể chạy single-user local nhưng **vẫn ghi actor** (username OS + profile nội bộ).

---

## 4. Integrity rules trước khi xuất báo cáo “chính thức”

Checklist cứng:

1. Mọi ImageAsset trong report có SHA-256.
2. Không còn `hash_mismatch` chưa được supervisor resolve.
3. Mọi Finding `confirmed` có ≥ 1 evidence ref còn tồn tại.
4. Audit có `report_exported` sau khi sinh file.
5. Ghi rõ giới hạn kỹ thuật (FS hỗ trợ, carving confidence).

Nếu fail checklist → chỉ cho xuất **Draft watermark**, không xuất **Official**.

---

## 5. Báo cáo giám định

### 5.1. Cấu trúc đề xuất

1. **Trang bìa:** tên lab, số case, ngày, giám định viên.
2. **Tóm tắt nhiệm vụ / câu hỏi giám định** (text nhập).
3. **Mô tả evidence:** nguồn, serial, image path, format, size, hash.
4. **Phương pháp:** tool version, module đã chạy (FS parse, carving pack version).
5. **Findings:** bảng + mô tả; mỗi finding có phụ lục provenance.
6. **Timeline tóm tắt** (optional).
7. **Kết luận kỹ thuật** (do người nhập — không auto sinh khẳng định pháp lý).
8. **Appendix A:** custody log rút gọn.
9. **Appendix B:** danh sách file exported (path, hash, nguồn).
10. **Appendix C:** bad sector / lỗi acquisition (nếu có).

### 5.2. Định dạng xuất

| Format | Ưu tiên |
|--------|---------|
| PDF | P0 |
| HTML | P0/P1 |
| JSON manifest (machine-readable) | P0 |
| CSV findings | P1 |
| STIX/CFO subset | P2 — liên kết `STIX_for_digital_forensics/` |

### 5.3. JSON manifest (ý tưởng)

```json
{
  "case_id": "...",
  "exported_at": "...",
  "tool": { "name": "DiskForge", "version": "0.1.0" },
  "images": [{ "id": "...", "sha256": "...", "format": "dd" }],
  "findings": [
    {
      "id": "...",
      "title": "...",
      "status": "confirmed",
      "evidence": [
        { "type": "extracted_file", "sha256": "...", "source_offset": 123456, "fs_ref": "MFT 4242" }
      ]
    }
  ]
}
```

---

## 6. Retention & bảo vệ dữ liệu lab

- Thư mục case tách: `images/` (có thể external NAS), `exports/`, `reports/`, `db/`.
- Không upload cloud mặc định.
- Hướng dẫn xóa an toàn case khi hết thời hạn lưu trữ (P1 documentation).

---

## 7. Yêu cầu chức năng

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| CU-01 | Audit append-only cho sự kiện P0 | P0 |
| CU-02 | Hiển thị custody timeline trong UI | P0 |
| CU-03 | Export PDF official/draft | P0 |
| CU-04 | Export JSON manifest | P0 |
| CU-05 | Supervisor override có lý do | P1 |
| CU-06 | Hash-chain verify tool | P1 |
| CU-07 | STIX export | P2 |

---

## 8. Tham chiếu repo

- `Basic_Computer_Skills_for_Forensics/Forensic_Report_Template.pdf`
- STIX/CFO objects: Image, Investigator, Timeline, Crime Case
- Paper PST2022 về presentable evidence + knowledge graphs (hướng mở rộng)

---

## 9. Tiêu chí chấp nhận

- Sau một case mẫu: PDF official chứa hash image + ≥ 1 finding kèm provenance + appendix custody.
- Sửa tay một dòng audit (trong test) → hash-chain verify báo fail (nếu đã triển khai CU-06).
- Draft report có watermark rõ; official chỉ khi checklist pass.
