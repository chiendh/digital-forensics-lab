# 07 — Data Model & Architecture

## 1. Mục tiêu tài liệu

Chốt:

- Schema Case DB logic
- Ranh giới process/module trên Windows
- Hợp đồng dữ liệu giữa Acquisition → FS → Carving → Analysis → Report
- Hướng tích hợp thư viện mở (TSK, libewf, …) mà không khóa kiến trúc

---

## 2. Kiến trúc tổng thể

```text
┌──────────────────────────────────────────────────────────────┐
│                 DiskForge.UI (Windows Desktop)               │
│         Case · Device · Browser · Timeline · Report          │
└────────────────────────────┬─────────────────────────────────┘
                             │ async commands / jobs
┌────────────────────────────▼─────────────────────────────────┐
│                    DiskForge.AppHost                          │
│         Job queue · Progress · Cancellation · Policies       │
└───┬──────────────┬──────────────┬──────────────┬─────────────┘
    │              │              │              │
    ▼              ▼              ▼              ▼
Acquisition   FsEngine      CarveEngine     ReportEngine
    │              │              │              │
    └──────────────┴──────────────┴──────────────┘
                             │
                             ▼
                 CaseStore (SQLite) + FileStorage
                 images/  exports/  reports/  logs/
```

### 2.1. Quyết định MVP

| Quyết định | Lựa chọn | Lý do |
|------------|----------|-------|
| UI stack | Desktop Windows (WPF/WinUI/Qt — để team chọn sau) | Phù hợp lab giám định |
| Case DB | SQLite mỗi case (+ optional catalog DB) | Dễ copy case folder |
| Imaging | In-process hoặc worker process cùng máy | Tránh ghi nhầm; isolate crash |
| FS parse | Ưu tiên bọc TSK/libtsk hoặc tương đương | Time-to-value |
| Carving | Engine riêng + signature JSON | Tách khỏi FS |
| Mobile | Không có trong host này | Module sau |

### 2.2. Process isolation (khuyến nghị)

- `DiskForge.UI` + `AppHost` ổn định.
- Job nặng (hash toàn image, carve full disk) chạy **worker** riêng để UI không đơ; crash worker không mất Case DB.

---

## 3. Case folder layout

```text
Case_<id>/
  case.sqlite
  case.json                 # mirror metadata nhẹ
  images/
    <image_id>.dd
    <image_id>.dd.meta.json
  exports/
    <extract_id>_<name>
  reports/
    draft_...
    official_...
  work/
    indexes/
    thumbnails/
  logs/
    audit.ndjson            # optional mirror of DB audit
```

---

## 4. Schema logic (bảng chính)

### 4.1. Case & people

```text
cases(id, title, case_number, description, timezone, created_at, closed_at, status)
users(id, username, display_name, role)
case_members(case_id, user_id, role)
```

### 4.2. Evidence & acquisition

```text
evidence_sources(id, case_id, type, label, hardware_model, serial, notes)
acquisition_sessions(
  id, evidence_source_id, started_at, ended_at, status,
  blocker_type, blocker_model, bad_sector_policy, operator_id
)
image_assets(
  id, case_id, evidence_source_id, acquisition_session_id,
  path, format, size_bytes, md5, sha256, verify_status,
  acquired_at, tool_version
)
bad_sectors(id, acquisition_session_id, offset_or_lba, length, detail)
```

### 4.3. Partition & FS

```text
partitions(id, image_id, index_no, start_offset, length, scheme, type_guid_or_id)
volumes(id, partition_id, fs_type, label, serial, status)
fs_entries(
  id, volume_id, parent_id, path, name, is_dir, is_deleted,
  size, recoverability, overwrite_estimate, fs_ref_json, timestamps_json
)
```

### 4.4. Carving & extracts

```text
carved_objects(
  id, image_id, volume_id_nullable, rule_id, offset, length,
  confidence, truncated, sha256_nullable, duplicate_of_fs_entry_id
)
extracted_files(
  id, case_id, source_type, source_id, export_path,
  sha256, size, created_at, provenance_json
)
```

### 4.5. Analysis & report

```text
tags(id, case_id, name, color)
tag_links(tag_id, object_type, object_id)
notes(id, object_type, object_id, body, author_id, created_at)
findings(id, case_id, title, body, status, created_by, confirmed_by, confirmed_at)
finding_evidence(finding_id, object_type, object_id)
timeline_events(
  id, case_id, ts_utc, ts_source, title, description, confidence
)
timeline_event_links(event_id, object_type, object_id)
audit_events(
  id, case_id, ts_utc, actor_id, event_type, object_refs_json,
  details_json, tool_version, prev_hash, entry_hash
)
report_exports(
  id, case_id, kind, path, created_at, sha256, is_official
)
```

---

## 5. Hợp đồng job nội bộ (ý tưởng API)

```text
AcquireDisk(request) -> AcquisitionSessionId
VerifyImage(image_id) -> verify_status
ParsePartitions(image_id) -> partition[]
ParseVolume(volume_id) -> fs_entry_count
ExtractFsEntry(fs_entry_id) -> extracted_file_id
Carve(scope, signature_pack_id) -> job_id
Search(case_id, query) -> hits[]
BuildReport(case_id, official: bool) -> report_export_id
```

Mọi job trả progress event: `percent`, `phase`, `message`, `errors[]`.

---

## 6. Provenance JSON (chuẩn chung)

Mọi `extracted_files.provenance_json` nên đủ để tái lập:

```json
{
  "image_id": "...",
  "image_sha256": "...",
  "method": "fs_extract|carve",
  "volume_id": "...",
  "fs_ref": { "mft_record": 4242 },
  "byte_offset": 1048576,
  "length": 20480,
  "rule_id": "jpeg_v1",
  "engine": { "fs": "tsk-2025.x", "carver": "1.0.0" }
}
```

---

## 7. Tích hợp thư viện mở (gợi ý)

| Nhu cầu | Ứng viên | Ghi chú |
|---------|----------|---------|
| E01 | libewf / ewfnet | Repo đã nhắc ewf-tools |
| FS list/cat | The Sleuth Kit | Lab sẵn trong repo |
| Hash | OS crypto / OpenSSL | SHA-256 bắt buộc |
| Carving | Engine nội bộ hoặc foremost-like | Signature pack riêng |
| SQLite | sqlite3 | Case DB + sau này artifact |

**Biên pháp lý license:** kiểm tra license (IBM CPL/IPL cho TSK, LGPL/GPL cho một số lib) trước phân phối thương mại; tài liệu này không thay tư vấn pháp lý.

---

## 8. Bảo mật ứng dụng

- Case folder ACL: chỉ group lab.
- Không thực thi file extract.
- Preview office/pdf dùng renderer cô lập nếu có thể.
- Path traversal protection khi ghi export theo tên file từ FS độc hại.

---

## 9. Kiểm thử kiến trúc

| Loại | Ví dụ |
|------|-------|
| Unit | Partition parser, signature matcher |
| Fixture | USB DD trong repo / NIST subset |
| Golden hash | Image mẫu → SHA cố định |
| Regression carve | Số object theo rule_id trong biên dung sai số |
| Custody | Mọi acquire tạo đủ audit events |

---

## 10. Liên kết

- Roadmap triển khai theo schema này → [08](08_MVP_Roadmap_and_Backlog.md)
- CFO/STIX mapping tương lai: `x-image`, `x-disk-partition`, `x-timeline`, `x-crime-case`
