# 02 — Lab → Product Mapping

Ánh xạ thư mục lab trong repository sang: **kiến thức**, **fixture/test**, **module App 1 (Disk)** hoặc **App 2 (Mobile)**, và **tiêu chí đánh giá**.

---

## 1. App 1 — Disk Recovery

| Lab / folder | Know-how chính | Fixture / dữ liệu | Module PRD Disk | Eval gợi ý |
|--------------|----------------|-------------------|-----------------|------------|
| `Basic_Computer_Skills_for_Forensics/7_USB_Image_Acquisition*` | Imaging USB, hash | `dd_image/usb_dd.7z` | 02 Acquisition | Hash khớp tool độc lập |
| `…/6_Sleuth_Kit_Tutorial*` | Partition, `fls`/`icat` | — | 03 Filesystem | Liệt kê đúng cây file |
| `…/9_Data_Carving*` + `file_carving/` | Signature carving | `usb_image/120M.7z`, manual carving set | 04 Carving | JPEG/PDF carved ≥ threshold |
| `…/8_*Evidence_search*` + `regex/` | Pattern search | CSV/HTML/PDF samples | 05 Search | Regex email/IP hit |
| `…/10_Steganography*` | Stego (mở rộng) | BMP hidden samples | Roadmap B/C | Detect known stego |
| `…/Forensic_Report_Template.pdf` | Cấu trúc báo cáo | — | 06 Report | Section coverage checklist |
| `NIST_Data_Leakage_Case/` | Registry, EVTX, MFT, USN, VSS, recycle, carving | NIST CFReDS images (ngoài) | 03–05 + B artifacts | Timeline đa nguồn |
| `P2P_Leakage/` | MFT/USN timeline, file signature, uTorrent | Case image riêng | 04–05 | Signature + timeline |
| `Illegal_Possession_Images/` | Recover + stego + HTTP/FTP | DFRWS Rhino USB | 03–04 + network later | Multi-method recovery |
| `Illegal_File_Transferring_Memory_Forensics/` | Memory timeline | — | Roadmap C (memory) | Optional |
| `Networking_Forensics/` | PCAP skills | `lab_files/*.pcap` | Out of MVP Disk | Optional App add-on |
| `STIX_…/Illegal_Possession_Images/` | CFO JSON mẫu | `*.json` | 07 + academic 04 | Export isomorphic subset |
| `Help/tool-install*.sh` | Tool baseline | — | Engineering | Reproducible lab env |

---

## 2. App 2 — Mobile / IoT Recovery

| Lab / folder | Know-how chính | Fixture / dữ liệu | Module PRD Mobile | Eval gợi ý |
|--------------|----------------|-------------------|-------------------|------------|
| `Android10/` Labs 0–3 | Device/image/settings | Digital Corpora Android 10 | 01–02 ingest + device dossier | Đúng model/OS fields |
| `Android10/` Lab 4 | App lifecycle / sandbox | `data/data/<pkg>` | 03 App inventory | Đếm package + path |
| `Android10/` 5.1.x AOSP | SMS/Contacts/Calendar | Image corpora | 04 Native parsers | Conversation/contact counts vs gold |
| `Android10/` 5.2.x GMS | Messaging/Dialer/Maps/Photos | **`gmm_myplaces.db`** script | 04–05 Maps/Photos | Lat/lon decode; CSV rows |
| `Android10/` 5.3.x | Kik/TextNow/WhatsApp | Image corpora | 06 Third-party packs | Thread+attachment listing |
| `Android10/` Lab 6 | Rooting implications | — | 02 Acquisition modes | Capability matrix badge |
| `Android10/sqlite_tutorial/` | SQL + EAV contacts model | Teaching DBs | 03 SQLite explorer | JOIN reconstruct contact |
| `iOS/` Labs 0–9 | Native iOS artifacts | Digital Corpora iOS 13 | 04–05 iOS packs | Messages/Safari/Photos |
| `iOS/` Labs 10–12 | KnowledgeC, Health, Location | Corpora | 05 Behavioral/location | Timeline density |
| `iOS/` Labs 13–14 | Cellebrite / AXIOM parity | — | 08 Eval matrix | Gap table vs commercial |
| `iOS/` Lab 15 | Jailbreak | — | 02 Trust/scope | Warning + scope flag |
| `Echo_Device/` | Chip-off eMMC, accounts, Wi-Fi, carve | `lab_data/Lab_4_*` | 07 IoT ingest | build.prop; WifiConfig; carved JPG count |
| `Eufy_Doorbell/` | Chip-off, P2P/sec logs, camera/SQLite dirs | eMMC link ngoài | 07 IoT | Dir inventory + log parse |
| `Drone_DJI_Mavic_Air/` | Controller + microSD GPS/cache | VTO Labs dataset | 07 Drone | GPS↔image correlate |
| `AI4Forensics/` | Entity/KG/STIX assist | Phishing notebooks | Shared AI later | P/R entity; human confirm |
| CTF repo ngoài (`digital-forensics-lab-p2`) | Multi-phone group crime | Cellebrite CTF 2022 | Multi-device roadmap | Cross-device graph |

---

## 3. Ma trận “lab dạy gì / app cần thêm gì”

| Lab làm tốt | App bắt buộc bổ sung |
|-------------|----------------------|
| Phân tích thủ công image có sẵn | Adapter acquisition + custody UI |
| Walkthrough từng app (PPT) | Parser pack versioned + CI schema drift |
| Rèn SQLite | UI conversation/map/gallery + generic explorer |
| Kể chuyện chip-off | Ingest/mount/hash image; không dạy tháo chip trong app |
| So sánh Cellebrite/AXIOM (iOS) | Capability matrix trung thực |
| Carving Echo (foremost/scalpel) | Carver tích hợp + confidence + dedupe |
| STIX JSON mẫu | Export subset có PreDIE gate |

---

## 4. Thứ tự nghiên cứu khuyến nghị (học thuật → PRD)

1. Charter 2 app (xong)  
2. Disk PRD (xong)  
3. Mobile artifact catalog từ Android/iOS/Echo (PRD Mobile)  
4. Evaluation framework dùng chung  
5. STIX/PreDIE dùng chung  
6. Fixtures: chọn 1 USB DD + 1 Android corpora subset + 1 Echo userdata sample cho golden tests  

Chi tiết backlog mở: [05_Shared_Research_Backlog.md](05_Shared_Research_Backlog.md).
