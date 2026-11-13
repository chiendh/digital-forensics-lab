# 02 — Acquisition Modes & Trust Model

## 1. Mục tiêu

Mô hình hóa **cách evidence mobile/IoT được đưa vào case**, mức độ bao phủ, và **badge tin cậy** hiển thị cho giám định viên / nghiên cứu viên.

App **MVP ưu tiên import ảnh/dump có sẵn** (đúng tinh thần lab Digital Corpora / chip-off image). Live acquisition có thể giai đoạn B.

---

## 2. Ma trận acquisition

| Mode | Mô tả | Phạm vi dữ liệu | Trust badge | MVP |
|------|-------|-----------------|-------------|-----|
| `logical_backup` | ADB backup / iTunes/Finder backup / vendor logical | App data được backup cho phép | Medium | P0 import |
| `filesystem_dump` | `adb pull` userdata / agent FS listing | Rộng hơn logical nếu quyền đủ | Medium–High | P0 import |
| `physical_image` | bit-for-bit userdata/eMMC (đã có file) | Cao nhất trên media | High | P0 import |
| `chip_off_emmc` | Ảnh sau chip-off (Echo/Eufy) | Toàn chip; cần partition map | High + “physical destructive process” note | P0 import |
| `microsd_image` | Thẻ drone/camera | Media + GPS logs | High | P0 import |
| `cloud_export` | Takeout / user data download | Phụ thuộc nhà cung cấp | Medium; provenance cloud | P1 |
| `live_adb` / `live_lockdown` | Thu thập trực tiếp thiết bị | Phụ thuộc unlock/debug | Variable | P2 |

---

## 3. Android: rooting (lab 6)

| Không root | Có root / privileged |
|------------|----------------------|
| Giới hạn thư mục accessible | Đọc sâu `data/data`, có thể keystore tùy case |
| Phù hợp nhiều case dân sự/lab | Cần ghi nhận phương pháp; có thể ảnh hưởng tính toàn vẹn nếu thao tác sai |

**Yêu cầu sản phẩm:**  
`AcquisitionSession.privilege_level = standard|root|unknown`  
Cảnh báo UI khi parser cần path chỉ có trên root nhưng image là logical hẹp.

---

## 4. iOS: jailbreak (lab 15)

| Không jailbreak | Jailbreak image |
|-----------------|-----------------|
| Backup/logical phổ biến | File system rộng hơn |
| Một số DB mã hóa/khó | Vẫn không giải mọi Data Protection keys |

Badge: `jailbroken=yes|no|unknown` + ghi chú lab method.

---

## 5. Chip-off / eMMC (Echo, Eufy)

Lab dạy teardown; **app chỉ**:

1. Nhận file image (+ optional userdata partition).  
2. Hash + partition table.  
3. Mount/parse read-only (ext4, raw regions).  
4. Chạy artifact + carving.  

Ghi `destructive_acquisition=true` trong custody (chip-off là quy trình phá hủy vật lý — metadata quan trọng cho hồ sơ).

---

## 6. Pre-flight & custody fields

```text
device_type: android|ios|iot_echo|iot_eufy|drone|other
model, os_version, identifiers[]
acquisition_mode
privilege_level
jailbroken / rooted flags
source_lab_reference   # e.g. Digital Corpora Android 10
image_hashes
operator_id, started_at, tool_version
```

---

## 7. Yêu cầu chức năng

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| MA-01 | Import thư mục dump / archive / raw image | P0 |
| MA-02 | Hash SHA-256 + verify | P0 |
| MA-03 | Hiển thị trust/capability badges | P0 |
| MA-04 | Partition map cho eMMC/physical | P0 |
| MA-05 | Ghi lab/dataset citation trong case | P0 (học thuật) |
| MA-06 | Live ADB acquire | P2 |
| MA-07 | iTunes backup decrypt (với password hợp lệ) | P1 |

---

## 8. Tiêu chí chấp nhận

- Import Android Corpora-style tree hoặc image; badge mode đúng.  
- Import Echo userdata sample; hiện partition/files.  
- Custody có đủ mode + hash + privilege flags.  
