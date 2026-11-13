# 04 — Android Artifact Packs

Dựa trên lộ trình lab `Android10/` (Pixel 3 / Android 10, Digital Corpora). Tên file DB cụ thể trong nhiều bài nằm ở PPT; dưới đây là **catalog điều tra + yêu cầu sản phẩm**. Đường dẫn tường minh sẽ bổ sung vào backlog học thuật (R-02) khi đối chiếu corpora.

---

## 1. Thứ tự pack (theo lab pedagogy)

```text
Device/System → App lifecycle awareness → AOSP native → GMS → Third-party → Root implications
```

---

## 2. Pack P0 — Device & system

| Artifact nhóm | Câu hỏi điều tra | Ghi chú lab |
|---------------|------------------|-------------|
| Image/device identity | Máy gì, OS nào? | Labs 0–2 |
| System settings | Ngôn ngữ, timezone, accounts? | Lab 3 |
| Network basics | Wi-Fi đã biết (nếu có trong dump) | Cross-ref Echo later |

**Output UI:** Device dossier panel.

---

## 3. Pack P0 — AOSP Communications

| Lab | Nhóm | Năng lực app tối thiểu |
|-----|------|------------------------|
| 5.1.1 Messaging | SMS/MMS threads | Thread list, message body, timestamps, participants |
| 5.1.2 Contacts | Danh bạ | Name, phones, emails; hiểu EAV-style storage |
| 5.1.3 Calendar | Sự kiện | Title, start/end, attendees nếu có |

**UI:** Conversation view (messaging), contact card, calendar list → timeline hooks.

---

## 4. Pack P0/P1 — GMS

| Lab | Nhóm | Năng lực |
|-----|------|----------|
| 5.2.1 Messaging | Google messaging | Tương tự threads (schema khác AOSP) |
| 5.2.2 Dialer | Call logs | Number, direction, duration, ts |
| 5.2.3 Maps | Saved places | **Đã có script:** `gmm_myplaces.db` / `sync_item` |
| 5.2.4 Photos | Media library | File path, media store metadata, optional EXIF |

### 4.1. Maps decoder (chuẩn hóa từ repo script)

```text
Input:  gmm_myplaces.db → sync_item
Fields: corpus, key_string, timestamp_ms, latitude, longitude, is_local, sync_item BLOB
Transform:
  - ts_utc = epoch_ms / 1000
  - lat = latitude / 1e6 ; lon = longitude / 1e6
  - blob preview utf-8/hex
Output: place facts + CSV/JSON + map pins
```

Đây là **golden micro-benchmark** cho App2.

---

## 5. Pack P1 — Third-party starters

| Lab | App | MVP expectation |
|-----|-----|-----------------|
| 5.3.1 | Kik | Account + message list nếu DB plaintext trên image |
| 5.3.2 | TextNow | Call/SMS-like records nếu đọc được |
| 5.3.3 | WhatsApp | Starter: phát hiện DB (`msgstore` class), thông báo nếu encrypted; parse được bản không mã hóa / đã decrypt sẵn trên dump nghiên cứu |

**Công bố trung thực:** “hỗ trợ khi DB readable trên dump phiên bản đã kiểm thử”.

---

## 6. Root pack (metadata)

Lab 6 không phải parser nội dung; sản phẩm ghi nhận **ảnh hưởng coverage** và checklist “các path chỉ xuất hiện khi privileged”.

---

## 7. Yêu cầu chức năng

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| AN-P01 | Device dossier từ dump Android 10 style | P0 |
| AN-P02 | AOSP messages+contacts+calendar normalize | P0 |
| AN-P03 | Call log GMS/AOSP | P0 |
| AN-P04 | Maps myplaces pack = script parity | P0 |
| AN-P05 | Photos media index | P1 |
| AN-P06 | WhatsApp/Kik/TextNow detection + best-effort | P1 |
| AN-P07 | Version matrix document per pack | P0 (học thuật) |

---

## 8. Acceptance

- Places pack: số row và tọa độ khớp output script mẫu trên cùng DB.  
- ≥1 conversation thread render từ AOSP messaging trên corpora.  
- Coverage table Android điền % pack P0.  
