# 05 — Search, Timeline & Analysis Workbench

## 1. Mục tiêu module

Giúp giám định viên **làm việc trên kết quả đã thu thập/phục hồi**:

- Tìm kiếm nhanh theo tên, hash, nội dung, regex
- Gắn tag / ghi chú / đánh dấu finding
- Dựng timeline nhẹ từ timestamp filesystem + sự kiện thủ công
- Chuẩn bị dữ liệu cho báo cáo ([06](06_Custody_Audit_and_Reporting.md))

Module này **không** phải AI tự kết luận; là bàn làm việc điều tra.

---

## 2. Search

### 2.1. Lớp tìm kiếm

| Lớp | Phạm vi | Ưu tiên |
|-----|---------|---------|
| Metadata index | Tên file, path, size, ext, deleted flag | P0 |
| Export content | Full-text trên file đã extract (text, UTF-8/UTF-16) | P0 |
| Hash lookup | MD5/SHA-256 của entry/export | P0 |
| Regex / pattern | Email, IP, CCCD/CMND-like, keyword list | P0/P1 |
| Raw image scan | strings trên scope chọn | P1 |
| YARA-like rules | Malware/triage packs | P2 |

### 2.2. UX

- Ô search toàn cục + filter facet: partition, type, deleted, carved, tagged.
- Kết quả click → mở preview hoặc hex.
- Lưu search query vào case (tái lập).

### 2.3. Hiệu năng

- Index metadata ngay khi parse FS.
- Content index **lazy** theo file đã extract hoặc theo hàng đợi background.
- Raw scan cảnh báo thời gian trên image lớn.

---

## 3. Tagging & findings

```text
Tag: string có màu / nhóm (ví dụ: Relevant, Key, Contraband, Needs-Review)
Note: rich text ngắn gắn FsEntry | CarvedObject | ExtractedFile
Finding: “assertion” mức case — gồm 1..n evidence refs + mô tả analyst
```

**Finding** là đơn vị đưa vào báo cáo (không phải mọi file extract).

Yêu cầu:

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| AN-01 | Tag/untag nhiều object | P0 |
| AN-02 | Note theo object | P0 |
| AN-03 | Tạo Finding từ selection | P0 |
| AN-04 | Finding status: draft/confirmed/rejected | P0 |
| AN-05 | Bookmark hex offset | P1 |

---

## 4. Timeline

### 4.1. Nguồn sự kiện MVP

| Nguồn | Sự kiện ví dụ |
|-------|----------------|
| FsEntry timestamps | Created / Modified / Accessed / Entry-modified (nếu có) |
| AcquisitionSession | Image acquired |
| Analyst manual | “USB plugged (theo lời khai)” — soft event |
| CarvedObject | Time = discovery time (không phải thời điểm file gốc) — phải gắn nhãn khác |

### 4.2. Chuẩn hóa TimelineEvent

```text
TimelineEvent
  id, case_id
  ts_utc
  ts_source          # SI_mtime | FN_ctime | manual | acquisition | carved_discovered
  title, description
  actor_hint?        # optional free text MVP
  object_refs[]      # FsEntry / ExtractedFile / ...
  confidence         # high nếu từ FS; low nếu suy diễn
```

### 4.3. UX timeline

- Trục thời gian zoom theo ngày/giờ.
- Filter theo nguồn timestamp, tag, partition.
- Cảnh báo: **MACB không phải sự thật tuyệt đối** (anti-forensics, timezone, USB copy).

### 4.4. Mở rộng giai đoạn B

- USN Journal, $LogFile, Prefetch, LNK, JumpList, Event Log — ingest thành TimelineEvent.
- Tham chiếu NIST labs trong repo.

---

## 5. Preview & viewers

| Loại | MVP |
|------|-----|
| Text | Có (encoding detect cơ bản) |
| Hex | Có |
| Ảnh | JPEG/PNG/GIF thumbnail |
| PDF | Trang đầu hoặc external open |
| Office | Metadata tối thiểu; mở ngoài tùy chọn |
| Video | P1 — thumbnail / external player |

---

## 6. Analysis workflow gợi ý (trong sản phẩm)

```text
1. Orientation: xem partition map + top-level folders
2. Triage search: tên nhạy cảm, ext media, keyword
3. Recover deleted candidates (FS)
4. Carve unallocated nếu thiếu
5. Tag + Findings
6. Arrange Timeline cho câu chuyện điều tra
7. Freeze confirmed findings → Report
```

UI có thể có checklist “Investigation steps” theo case template (tùy chọn P1).

---

## 7. Giới hạn

- Không xây graph quan hệ phức tạp kiểu STIX full ở MVP (export subset ở B).
- Không auto-correlate đa image trừ khi cùng case và analyst nối thủ công.
- Timezone: lưu UTC; hiển thị theo timezone case setting.

---

## 8. Tham chiếu repo

- Evidence search / pattern match labs
- NIST timeline-oriented labs (MFT, USN, WebHistory)
- P2P leakage timeline teaching materials

---

## 9. Tiêu chí chấp nhận

- Search theo tên file trả kết quả < 2s trên index metadata case mẫu.
- Tạo được Finding gắn 2+ evidence refs và thấy trên report draft.
- Timeline hiển thị sự kiện FS + acquisition; carved events tách màu/nhãn.
