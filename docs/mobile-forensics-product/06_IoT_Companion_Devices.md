# 06 — IoT Companion Devices (Echo, Eufy, DJI)

Thiết bị “cạnh mobile” trong repo: loa/màn hình thông minh, chuông cửa, drone. App2 hỗ trợ **ingest & phân tích image**, tái sử dụng carving/search từ App Disk.

---

## 1. Amazon Echo Show 8 (`Echo_Device/`)

### 1.1. Acquisition context

- Chip-off eMMC; full + userdata images (link ngoài; `disk_images/readme.txt`)  
- Fire OS / Android-like (`build.prop`: model `AEOAT`, device `athena`, Android 9)

### 1.2. Artifact groups từ `lab_data/`

| Nhóm | Ví dụ file | Parser / UI |
|------|------------|-------------|
| Device specs | `build.prop`, settings XML | Dossier |
| Accounts | `accounts_ce.db`, `accounts_de.db`, userlist XML | Account facts |
| Recent tasks / snapshots | `recent_tasks/*.xml`, `snapshots/*` | Activity timeline |
| Daily logs | `daily/<epoch_ms>` | Log viewer |
| Network | `WifiConfigStore.xml`, `bt_config.conf` | Wi-Fi/BT facts |
| Web | Chrome-style `History` | Browser module reuse |
| Phone/SIP | carved XML → call JSON | Call-like facts |
| Media carved | `rec_jpg_foremost/` | Gallery + offsets |
| Playback DBs | mixtape / playbackhistory | Media history |
| Bulk features | `bulkext/` | Optional import bulk_extractor output |

### 1.3. Carving note

Lab dùng **foremost / Scalpel / PhotoRec**. App nên gọi chung CarveEngine (App1) trên vùng raw + gắn provenance.

---

## 2. Eufy Doorbell + HomeBase 2 (`Eufy_Doorbell/`)

### 2.1. Workflow lab

Scenario → chip-off → mount → P2P logs → daily/sec logs → camera dir → SQLite dir.

### 2.2. Product capabilities

| Năng lực | Mô tả | Ưu tiên |
|----------|-------|---------|
| eMMC ingest | Như Echo | P0 |
| Directory presets | Nhận diện camera/sqlite/log folders | P1 |
| P2P / security log parse | Timeline events | P1 |
| Footage index | File media + timestamps | P1 |
| SQLite packs | Khi biết schema HomeBase | P2 |

---

## 3. DJI Mavic Air (`Drone_DJI_Mavic_Air/`)

### 3.1. Dual source

1. **Controller (mobile)** logical artifacts  
2. **microSD** raw / EnCase container  

### 3.2. Product capabilities

| Năng lực | Ưu tiên |
|----------|---------|
| Import microSD image + hash | P0 |
| Media gallery + EXIF/GPS | P0 |
| GPS track timeline | P1 |
| Correlate track ↔ cached images | P1 |
| Import controller mobile dump into same case | P1 |
| EnCase container read | P2 |

Lab đang draft — công bố “experimental”.

---

## 4. Shared IoT requirements

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| IOT-01 | Physical/chip-off image import + partitions | P0 |
| IOT-02 | build.prop / device XML dossier | P0 |
| IOT-03 | Wi-Fi/BT config parse (Echo patterns) | P0 |
| IOT-04 | Reuse disk carving on IoT images | P0 |
| IOT-05 | Eufy folder presets + log timeline | P1 |
| IOT-06 | Drone GPS↔image correlation view | P1 |

---

## 5. Acceptance

- Echo lab_data: đọc được model từ build.prop; parse SSID list; mở History DB nếu có.  
- Chạy carving hoặc index sẵn `rec_jpg_foremost` với offset provenance.  
- Case có thể chứa cùng lúc điện thoại + IoT image (multi-source).  
