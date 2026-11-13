# 02 — Acquisition & Imaging

## 1. Mục tiêu module

Cho phép giám định viên **tạo hoặc tiếp nhận** bản sao bit-level (hoặc gần bit-level) của nguồn evidence, kèm:

- Ghi nhận phần cứng / định danh nguồn
- Chế độ chống ghi (write-blocker cứng hoặc mềm)
- Hash toàn image
- Nhật ký lỗi (bad sector, retry)
- Bản ghi custody

---

## 2. Nguồn evidence được hỗ trợ

| Loại nguồn | Ví dụ | Hành vi sản phẩm |
|------------|-------|------------------|
| **Internal disk** | HDD/SSD gắn SATA/NVMe trong máy lab hoặc máy đích được phép | Liệt kê disk, imaging sang file trên storage riêng |
| **Removable** | USB flash, HDD box, thẻ SD qua reader | Ưu tiên gắn qua hardware write-blocker |
| **Existing image file** | `.dd`, `.raw`, `.img`, `.e01` | Import + verify hash; không imaging lại |

**Quy tắc lưu trữ:** Image luôn ghi lên **ổ/volume không phải nguồn evidence**. UI phải chọn đích rõ ràng và cảnh báo nếu đích trùng disk nguồn.

---

## 3. Write protection

### 3.1. Hardware write-blocker (khuyến nghị hiện trường)

- UI có checklist: “Đã nối qua write-blocker?” (bắt buộc xác nhận trước khi acquire removable/internal khi policy lab yêu cầu).
- Ghi vào `AcquisitionSession.blocker_type = hardware` + model (text).

### 3.2. Software / OS controls (bổ sung, không thay thế hoàn toàn)

- Trên Windows: phát hiện volume, ưu tiên mở handle read-only.
- Cảnh báo lớn nếu volume đang mounted read-write.
- Không tự động format, không chạy chkdisk ghi sửa trên nguồn.

### 3.3. Policy

| Tình huống | Mặc định |
|------------|----------|
| Removable tại hiện trường | Bắt buộc xác nhận blocker |
| Import image có sẵn | Không cần blocker; verify hash |
| Lab training mode | Cho phép bỏ qua với lý do ghi vào audit |

---

## 4. Quy trình imaging

```text
1. Chọn Case
2. Discover devices (disk list: model, serial, size, bus, removable?)
3. Chọn source disk/volume
4. Chọn đích file image + định dạng (DD / E01)
5. Chọn thuật toán hash (SHA-256 bắt buộc; MD5 tùy chọn song song)
6. Pre-flight checks (đích đủ chỗ, nguồn ≠ đích, blocker ack)
7. Stream copy + progress + throughput
8. Xử lý bad sector theo policy (pad zeros / skip + log)
9. Finalize: ghi metadata sidecar + custody entry
10. Optional: verify bằng re-hash đọc lại image
```

### 4.1. Bad sector policy

| Policy | Mô tả | Khi dùng |
|--------|-------|----------|
| `pad_zero` | Ghi block 0x00 vào chỗ đọc lỗi, log LBA | Mặc định — giữ kích thước/layout |
| `abort_on_error` | Dừng khi lỗi đọc | Khi lab yêu cầu image “sạch” hoàn toàn |
| `retry_then_pad` | Retry N lần rồi pad | Ổ già / USB lỗi |

Mỗi bad sector ghi: LBA (hoặc offset), số byte, thời điểm, kết quả retry.

### 4.2. Đa luồng & hiệu năng

- Đọc tuần tự theo block lớn (ví dụ 1–8 MiB) để tối ưu HDD; SSD có thể tăng queue nhưng vẫn ghi tuần tự ra file.
- Progress: byte đã đọc / tổng, tốc độ, ETA, số bad sector.
- Cho phép **pause/resume** nếu định dạng và triển khai cho phép (DD resume theo offset đã ghi; E01 phức tạp hơn — có thể để giai đoạn B).

---

## 5. Định dạng image

### 5.1. DD / RAW (P0)

- File đơn hoặc split (`.001`, `.002`…) tùy chọn.
- Ưu điểm: đơn giản, tương thích cao.
- Nhược điểm: không nén; metadata phải lưu sidecar.

**Sidecar tối thiểu (JSON hoặc DB row):**

```json
{
  "case_id": "...",
  "source_serial": "...",
  "source_size_bytes": 0,
  "image_path": "D:/Images/case1.dd",
  "format": "dd",
  "block_size": 65536,
  "hashes": { "sha256": "...", "md5": "..." },
  "acquired_at": "2026-11-13T10:00:00Z",
  "tool_name": "DiskForge",
  "tool_version": "0.1.0",
  "blocker": { "type": "hardware", "model": "..." },
  "bad_sectors": []
}
```

### 5.2. E01 / EWF (P0 hoặc P1 sớm)

- Nén, metadata nhúng, phổ biến trong giám định.
- Có thể dựa trên libewf / ewf-tools (tham chiếu repo Tools).
- Trường metadata: case number, examiner, description, hash.

### 5.3. AFF (P2)

- Để giai đoạn sau khi DD+E01 ổn định.

---

## 6. Import image có sẵn

1. Chọn file / bộ split.
2. Nhận diện format.
3. Tính hoặc nhập hash người dùng cung cấp.
4. Verify (có thể background).
5. Gắn vào Case như `ImageAsset` với `acquisition_type = imported`.

Nếu verify fail: cho mở **ở chế độ cảnh báo** (read-only analysis) nhưng **không cho đóng báo cáo “đạt chuẩn custody”** trừ khi supervisor override có lý do.

---

## 7. Device discovery (Windows)

Thông tin tối thiểu mỗi disk:

- Index / DeviceId
- Model, Serial, Firmware
- Size, Sector size
- Bus type (USB, SATA, NVMe…)
- Removable flag
- Partition layout tóm tắt (nếu đọc được mà không ghi)

**Cảnh báo đặc biệt:** BitLocker / encrypted volumes — phát hiện và ghi chú; MVP có thể chỉ image bit-level mà chưa decrypt (decrypt = giai đoạn sau + yêu cầu key/password hợp pháp).

---

## 8. Yêu cầu chức năng chi tiết

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| ACQ-01 | Liệt kê disk vật lý trên Windows | P0 |
| ACQ-02 | Imaging nguồn → DD với SHA-256 | P0 |
| ACQ-03 | Checklist write-blocker + ghi audit | P0 |
| ACQ-04 | Bad sector log + pad_zero | P0 |
| ACQ-05 | Import DD và verify hash | P0 |
| ACQ-06 | Imaging/import E01 | P0/P1 |
| ACQ-07 | Split raw image | P1 |
| ACQ-08 | Pause/resume DD | P1 |
| ACQ-09 | Hardware blocker model catalog | P2 |
| ACQ-10 | AFF support | P2 |

---

## 9. Rủi ro & giảm thiểu

| Rủi ro | Giảm thiểu |
|--------|------------|
| Image nhầm lên nguồn | So sánh disk id nguồn/đích; confirm dialog |
| USB rút giữa chừng | Detect I/O error; giữ partial state + audit `failed` |
| Hash sai do file đích corrupt | Verify pass bắt buộc trước khi mark `acquired` |
| SSD có TRIM / hidden area | Ghi giới hạn: image phản ánh địa chỉable sectors; HPA/DCO để giai đoạn sau |
| Operator bỏ blocker | Policy bắt buộc ack; training mode tách biệt |

---

## 10. Tham chiếu lab trong repo

- `Basic_Computer_Skills_for_Forensics/7_USB_Image_Acquisition.pptx`
- `Help/Tool_installation.pptx`, `ewf-tools`, `hashdeep`
- Case dùng image NIST / Digital Corpora làm fixture kiểm thử

---

## 11. Tiêu chí chấp nhận (module)

- Tạo được DD từ USB mẫu; SHA-256 khớp khi hash lại bằng tool độc lập (hashdeep hoặc Get-FileHash).
- Import DD/E01 vào case; fail verify thì hiển thị trạng thái rõ.
- Custody log có đủ: ai, khi nào, nguồn, đích, hash, blocker ack.
