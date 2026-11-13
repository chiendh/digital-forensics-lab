# 06 — Artifact Catalog Starter (Android / iOS / IoT)

> Catalog **khởi đầu** phục vụ đánh giá học thuật và parser packs.  
> Nhiều path đầy đủ nằm trong PPT lab / Digital Corpora — các mục đánh dấu `verify_on_corpora` cần đối chiếu image trước khi đóng cứng vào engine.  
> Cập nhật liên tục theo backlog R-02 / R-03.

---

## 1. Android (lab `Android10/`, Pixel 3 / Android 10)

| Category | Locator gợi ý / đã xác nhận trong repo | Tables / notes | Pack | Confidence |
|----------|----------------------------------------|----------------|------|------------|
| App sandbox roots | `data/data/<package_id>/` | Inventory | inventory | **confirmed** (scripts) |
| Google Maps saved places | `gmm_myplaces.db` | `sync_item` (corpus, key_string, timestamp, lat, lon, is_local, sync_item) | maps | **confirmed** (script) |
| AOSP Messaging | typically `*/com.android.providers.telephony/databases/mmssms.db` | sms, pdu, … | aosp_messaging | verify_on_corpora |
| AOSP Contacts | typically `*/com.android.providers.contacts/databases/contacts2.db` | contacts EAV-like | aosp_contacts | verify_on_corpora |
| AOSP Calendar | typically `*/com.android.providers.calendar/databases/calendar.db` | events | aosp_calendar | verify_on_corpora |
| Call logs | telephony/contacts providers (OEM/GMS dependent) | calls | dialer | verify_on_corpora |
| GMS Messaging | package under Google messaging on image | — | gms_messaging | verify_on_corpora |
| Photos / MediaStore | Media provider DB + `DCIM/`/`Pictures/` | files + media rows | photos | verify_on_corpora |
| WhatsApp | `*/com.whatsapp/databases/msgstore.db` (+ crypt) | messages | whatsapp | verify_on_corpora + encryption caveat |
| Kik / TextNow | vendor packages on image | — | third_party | verify_on_corpora |
| build / device | `system/build.prop` (physical) | props | device | verify_on_corpora |

**Decode chuẩn Maps:** lat/lon ÷ 1e6; timestamp milliseconds → UTC (xem script `Android10/5_2_3_extract_myplaces_sync_items.py`).

---

## 2. iOS (lab `iOS/`, iOS 13.4.1)

| Category | Locator gợi ý | Notes | Pack | Confidence |
|----------|---------------|-------|------|------------|
| SMS/iMessage | `sms.db` (HomeDomain / wireless) | handles, message, chat | messages | verify_on_corpora |
| Contacts | `AddressBook.sqlitedb` / Contact DB | — | contacts | verify_on_corpora |
| Calendar | Calendar DB | — | calendar | verify_on_corpora |
| Safari | `History.db`, related plist | visits | safari | verify_on_corpora |
| Photos | Photos.sqlite + file tree | — | photos | verify_on_corpora |
| KnowledgeC | `knowledgeC.db` | app usage / interactions | knowledgec | verify_on_corpora |
| Health | `healthdb_secure.sqlite` (+ companions) | may be protected | health | verify_on_corpora |
| Location | cache/db variants per iOS version | — | location | verify_on_corpora |

Luôn gắn **iOS version** vào pack; 13.4.1 ≠ 17.x.

---

## 3. Echo Show (`Echo_Device/lab_data`) — confirmed filenames

| Category | Path pattern under lab_data | Notes |
|----------|-----------------------------|-------|
| Device | `Lab_4_1_1/build.prop` | model AEOAT / athena |
| Prefs | `deviceNameSharedPref.xml` | device display name |
| Accounts | `Lab_4_1_2/accounts_ce.db`, `accounts_de.db` | |
| Users | `userlist.xml`, `sorteduserlist.xml`, `0.xml` | |
| Recent tasks | `recent_tasks/*_task.xml` | |
| Snapshots | `snapshots/*.jpg`, `*.proto` | |
| Daily | `daily/<epoch_ms>` | |
| Wi-Fi | `Lab_4_1_3/WifiConfigStore.xml` | SSID/PSK |
| Bluetooth | `bt_config.conf` | |
| Web History | `Lab_4_2_1/History` | Chrome-style |
| Phone carved | `Lab_4_2_2/` JSON/XML | SIP/call |
| Carved JPG | `Lab_4_3_1/rec_jpg_foremost/` | foremost |
| Playback DBs | `Lab_4_3_2/` mixtape/playbackhistory | |
| Bulk extract | `Lab_4_1_2/bulkext/` | emails, GPS, … |

---

## 4. Eufy / DJI (directory-level until schemas locked)

| Device | Focus directories / sources | Status |
|--------|---------------------------|--------|
| Eufy HomeBase | Camera footage dir, SQLite dir, P2P logs, daily/sec logs | PPT-defined; verify on eMMC |
| DJI Mavic Air | Controller mobile extract + microSD image (raw/EnCase) | Draft lab; GPS + cached images |

---

## 5. Disk-side catalog (pointer)

Không lặp PRD Disk: xem modules carving signatures & NTFS trong `docs/digital-forensics-product/03–04`. NIST/P2P labs bổ sung USN/MFT/Prefetch cho giai đoạn B App1.

---

## 6. Quy tắc cập nhật catalog

1. Chỉ nâng `confirmed` khi đã thấy trên file thật hoặc script repo.  
2. Mỗi dòng `verify_on_corpora` cần issue/checklist khi mở image.  
3. Mọi parser pack phải trích dẫn **dong catalog + OS/app version**.  
