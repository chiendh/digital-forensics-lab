# Know-how định hướng tính năng: Digital Forensics & Mobile Forensics

> Tài liệu tổng quát giúp đội sản phẩm chuyển từ kiến thức điều tra (know-how) sang **phương hướng tính năng** cho phần mềm phục vụ digital forensic và mobile forensic.
>
> Nguồn tham chiếu chính trong repository này: lab Android/iOS, IoT (Echo, Eufy, DJI), case NIST/P2P, STIX/CFO, AI4Forensics, và danh mục tool mã nguồn mở.

---

## 1. Mục tiêu tài liệu

Tài liệu này trả lời bốn câu hỏi:

1. Phần mềm forensic cần **làm gì** theo vòng đời điều tra?
2. Domain know-how nào **bắt buộc** để thiết kế đúng tính năng?
3. Với **mobile forensic**, artifact và pipeline nào nên ưu tiên?
4. Làm sao biến know-how thành **roadmap tính năng** (MVP → nâng cao → AI)?

**Phạm vi:** định hướng sản phẩm / kiến trúc tính năng — không phải đặc tả API hay thuật toán production chi tiết.

---

## 2. Bản chất sản phẩm forensic

### 2.1. Sản phẩm không chỉ là “viewer dữ liệu”

Phần mềm digital/mobile forensic phải hỗ trợ chuỗi giá trị pháp lý:

| Giai đoạn | Mục tiêu nghiệp vụ | Hệ quả cho sản phẩm |
|-----------|--------------------|---------------------|
| **Identification** | Xác định nguồn evidence | Quản lý case, device inventory |
| **Preservation** | Bảo toàn tính toàn vẹn | Hash, write-blocker workflow, audit log |
| **Collection / Acquisition** | Thu thập image/logical dump | Hỗ trợ DD/E01/AFF, ADB backup, chip-off image, cloud export |
| **Examination** | Parse artifact thô | Parser filesystem, SQLite, plist, protobuf, log |
| **Analysis** | Liên kết sự kiện, dựng timeline | Correlation, timeline, graph quan hệ |
| **Presentation** | Báo cáo trình bày được trước tòa / nội bộ | Report template, export STIX/JSON/PDF, chain of custody |

Nếu thiếu bất kỳ giai đoạn nào, sản phẩm dễ dừng ở mức “tool kỹ thuật” thay vì “nền tảng điều tra”.

### 2.2. Hai lớp giá trị cần tách rõ

1. **Lớp kỹ thuật (engine):** acquisition helpers, parsers, carvers, hashers, timeline builders.
2. **Lớp nghiệp vụ (case workbench):** case, investigator, evidence object, action, timeline, report.

Know-how trong repo này cho thấy lớp nghiệp vụ có thể chuẩn hóa bằng mô hình **Cyber Forensic Objects (CFO / xSTIX)**: `x-image`, `x-action`, `x-timeline`, `x-crime-case`, v.v.

---

## 3. Know-how cốt lõi cần nắm trước khi thiết kế tính năng

### 3.1. Nguyên tắc bất biến

- **Không sửa evidence gốc.** Mọi phân tích chạy trên bản sao / image đã hash.
- **Mọi bước phải audit được.** Ai mở case, parse gì, export gì, lúc nào.
- **Evidence phải truy vết nguồn.** Artifact X đến từ file Y, offset Z, tool T, phiên bản V.
- **Thời gian là xương sống.** Hầu hết kết luận điều tra dựa trên timeline đa nguồn.
- **Artifact là “log chức năng” bị tái sử dụng.** App/OS ghi log để vận hành; investigator dùng lại để dựng hành vi người dùng.

### 3.2. Nhóm kỹ thuật nền (nên có trong engine)

| Kỹ thuật | Know-how | Tính năng sản phẩm gợi ý |
|----------|----------|--------------------------|
| **Imaging & integrity** | DD/E01, hash (MD5/SHA-1/SHA-256), verify | Import image, verify hash, re-hash on export |
| **Filesystem parse** | Partition table, NTFS/exFAT/APFS/ext4 | Browser phân vùng, tree file, deleted entries |
| **File signature / carving** | Magic bytes, header–footer, slack space | Carving module, recover deleted media |
| **Metadata extraction** | EXIF, Office metadata, LNK, Prefetch | Panel metadata + filter |
| **Structured DB parse** | SQLite (mobile), ESEDB, Windows Search | SQLite browser + artifact templates |
| **Log/event parse** | EVTX, syslog, app logs, USN/$MFT | Event viewer + timeline ingest |
| **Pattern search** | Regex, keyword, YARA-like | Global search across raw + parsed |
| **Steganalysis (tùy chọn)** | LSB / known tools | Detect hidden payloads in images |
| **Memory forensics (tùy chọn)** | Process, network, cmd history | Volatility-class plugins |
| **Network forensics (tùy chọn)** | PCAP, HTTP/SMTP/DNS | Session rebuild, file extract from traffic |

### 3.3. Mô hình dữ liệu điều tra (nên thiết kế sớm)

Gợi ý entity tối thiểu (bám CFO/STIX trong repo):

```text
CrimeCase
  └── Investigator(s)
  └── EvidenceSource (Device / Disk / Cloud / NetworkCapture)
        └── Image (format, hash, acquired_on, tool)
              └── Partition / Volume
                    └── File / ArtifactRecord
                          └── DerivedFact (Action, Visit, Message, Location…)
  └── Timeline (ordered Actions)
  └── Report / ExportPackage
```

Quan hệ quan trọng:

- `Image --evidence-of--> CrimeCase`
- `Action --indicated-by--> Indicator/Artifact`
- `Action --performed-by--> Actor/Account`
- `Timeline --reconstructs--> CrimeCase`

Thiết kế schema này sớm giúp tránh việc mỗi parser tự xuất CSV rời rạc, khó hợp nhất phân tích.

---

## 4. Know-how Mobile Forensic (trọng tâm sản phẩm)

### 4.1. Khác biệt so với computer forensics

| Khía cạnh | Computer | Mobile |
|-----------|----------|--------|
| Nguồn evidence | Disk image, memory, PCAP | Logical dump, filesystem dump, chip-off/eMMC, cloud |
| Lưu trữ chính | NTFS + registry + EVTX | SQLite + plist/protobuf + app sandbox |
| Rào cản | BitLocker, anti-forensics | Mã hóa full-disk, Secure Enclave/TEE, OEM lock, root/jailbreak |
| Artifact “nóng” | Browser, email, LNK, Prefetch | SMS/chat, call logs, location, photos, health, app-specific DBs |
| Vòng đời app | Ít sandbox hơn | Sandbox theo package; lifecycle app quyết định vị trí DB |

### 4.2. Pipeline mobile nên hỗ trợ

```text
[Device/Backup/Cloud]
        │
        ▼
 Acquisition (logical / filesystem / physical / chip-off import)
        │
        ▼
 Integrity (hash, case binding, custody log)
        │
        ▼
 Normalization (mount/extract → unified file tree)
        │
        ▼
 Artifact Decoding
   ├─ System settings / accounts / network
   ├─ AOSP/GMS or iOS native apps
   ├─ Third-party apps (WhatsApp, Kik, …)
   └─ Media + location + health sensors
        │
        ▼
 Correlation & Timeline
        │
        ▼
 Review UI + Tagging + Notes
        │
        ▼
 Report / STIX-JSON / export for review tools
```

### 4.3. Bản đồ artifact Android (ưu tiên triển khai)

Dựa trên lab Android 10 (Pixel 3) trong repository:

| Nhóm | Ví dụ điều tra | Hướng tính năng |
|------|----------------|-----------------|
| **Device & system** | Model, OS, settings, accounts, Wi-Fi | Device profile panel |
| **App inventory** | `data/data/<package>`, map package → app name | App catalog + Play metadata enrichment |
| **AOSP Messaging / Contacts / Calendar** | SQLite tables tin nhắn, danh bạ, sự kiện | Native parsers + conversation view |
| **GMS Messaging / Dialer / Maps / Photos** | Call logs, places, media library | Google artifact packs |
| **Third-party** | WhatsApp, Kik, TextNow | Plugin/parser marketplace theo app |
| **Rooting / elevated access** | Ảnh hưởng phạm vi thu thập | Acquisition mode indicator + capability matrix |
| **SQLite skills** | Schema discovery, deleted free pages | Generic SQLite explorer + undark-like recovery |

**Gợi ý kỹ thuật lặp lại trên Android:**

- Timestamp epoch milliseconds → UTC
- Tọa độ lưu integer (ví dụ ×1e6) → decimal degrees
- Cột BLOB cần decode thử UTF-8 / hex / protobuf
- Mỗi app = schema riêng; cần **parser pack** versioned theo app version

### 4.4. Bản đồ artifact iOS (ưu tiên triển khai)

Dựa trên lab iOS 13.4.1:

| Nhóm | Ý nghĩa điều tra | Hướng tính năng |
|------|------------------|-----------------|
| **Device & settings** | Nhận diện máy, cấu hình | Device dossier |
| **Messages / Contacts / Calendar** | Giao tiếp & lịch sử quan hệ | Conversation & contact graph |
| **Safari** | Duyệt web, lịch sử | Browser module |
| **Photos** | Media + metadata | Media gallery + EXIF/map |
| **KnowledgeC** | Thói quen dùng app / tương tác | Behavioral timeline (điểm mạnh iOS) |
| **Health** | Cảm biến sức khỏe | Health artifact pack |
| **Location** | Di chuyển, địa điểm quan trọng | Map timeline |
| **Jailbreak indicators** | Phạm vi & độ tin cậy thu thập | Trust/acquisition badge |
| **Tool parity** | So với Cellebrite / AXIOM | Gap analysis checklist trong QA |

### 4.5. IoT / thiết bị lai mobile (mở rộng sản phẩm)

Lab Echo Show, Eufy Doorbell, DJI cho thấy hướng mở rộng:

- **Chip-off / eMMC import:** nhận image thô, mount phân vùng, duyệt thư mục camera/SQLite/log
- **P2P / device logs:** dựng tương tác người dùng–thiết bị
- **Drone:** GPS track + cached images từ controller/SD

Đây là khác biệt cạnh tranh nếu sản phẩm nhắm “mobile + IoT companion devices”, không chỉ điện thoại.

---

## 5. Từ know-how → phương hướng tính năng sản phẩm

### 5.1. Nền tảng Case Workbench (bắt buộc)

1. **Case Management**
   - Tạo case, phân công investigator, mô tả kịch bản
   - Gắn nhiều evidence sources cho một case
2. **Evidence & Custody**
   - Đăng ký image/dump, hash lúc nhập, verify lại
   - Nhật ký thao tác (append-only)
3. **Unified Evidence Browser**
   - Cây file + tìm kiếm + xem hex/text/media
4. **Artifact Dashboard**
   - Số lượng message, call, location point, media, account…
5. **Timeline View**
   - Trộn đa nguồn, lọc theo actor/app/time range
6. **Reporting**
   - Template báo cáo, export PDF/HTML/JSON
   - Đính kèm hash và provenance từng finding

### 5.2. Engine Digital Forensics (máy tính / disk)

Ưu tiên theo độ tái sử dụng cao:

| Ưu tiên | Module | Lý do |
|---------|--------|-------|
| P0 | Image import + hash verify | Nền tảng pháp lý |
| P0 | Partition/FS browser | Điểm vào mọi case disk |
| P0 | Keyword/regex search | Nhu cầu điều tra phổ biến |
| P1 | Carving + file signature | Khôi phục dữ liệu bị xóa |
| P1 | Browser history / email parsers | Case mạng & rò rỉ dữ liệu |
| P1 | Windows artifact pack (Registry, EVTX, Prefetch, JumpList, USN, MFT) | Phủ case enterprise |
| P2 | Memory analysis | Case nâng cao |
| P2 | Network PCAP analysis | Case hybrid host+network |
| P2 | Steganography detect | Case chuyên biệt |

### 5.3. Engine Mobile Forensics (ưu tiên sản phẩm của bạn)

| Ưu tiên | Module | Ghi chú triển khai |
|---------|--------|--------------------|
| P0 | Logical/backup import (Android/iOS) | Hỗ trợ định dạng phổ biến trước; physical/chip-off nhận image có sẵn |
| P0 | App inventory + sandbox browser | Nhanh chóng định hướng điều tra |
| P0 | SQLite generic explorer | 80% artifact mobile đi qua đây |
| P0 | Native comms parsers (SMS/Call/Contacts) | Giá trị điều tra tức thì |
| P1 | Conversation UI (threads, attachments) | Trải nghiệm investigator |
| P1 | Location & Photos modules | Map + gallery liên kết timeline |
| P1 | iOS KnowledgeC / Health packs | Khác biệt so với tool chỉ đọc tin nhắn |
| P1 | Third-party chat starters (WhatsApp…) | Làm theo parser pack, versioned |
| P2 | Deleted SQLite recovery | Free list / WAL / undark-class |
| P2 | Cloud artifact ingest | Google Takeout, iCloud export (theo pháp lý) |
| P2 | IoT image analysis (Echo/Eufy/Drone) | Mở rộng thị trường |

### 5.4. Lớp phân tích nâng cao (AI / Knowledge Graph)

Từ AI4Forensics và paper Knowledge Graphs trong repo:

| Tính năng | Đầu vào | Đầu ra |
|-----------|---------|--------|
| Entity extraction | Log điều tra, email, chat | Actor, IP, file, URL, time |
| Relationship linking | Entities | Graph quan hệ |
| STIX/CFO export | Findings đã chuẩn hóa | Gói chia sẻ intelligence |
| Suspect profiling | Browser history / app usage | Hồ sơ hành vi (có disclaimer) |
| Assisted report drafting | Timeline + tagged findings | Bản nháp báo cáo |

**Nguyên tắc an toàn sản phẩm:** AI chỉ *đề xuất*; investigator xác nhận trước khi đưa vào báo cáo chính thức.

### 5.5. Phi chức năng (thường quyết định sản phẩm có dùng được trong lab thật)

- Hiệu năng trên image lớn (chục GB+)
- Xử lý incremental / background parsing
- Plugin architecture cho parser app mới
- Multi-user roles (viewer / analyst / admin)
- Air-gapped mode (lab không Internet)
- Reproducibility: cùng tool version → cùng kết quả parse
- Internationalization (UI + report), bao gồm tiếng Việt nếu thị trường nội địa

---

## 6. Gợi ý kiến trúc module (để team kỹ thuật bám)

```text
┌─────────────────────────────────────────────────────────┐
│                    Presentation Layer                    │
│  Case UI · Timeline · Map · Chat · Report Designer       │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│                 Analysis & Correlation                   │
│  Timeline engine · Graph · Search · AI assist (optional) │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│              Artifact Parsers (plugin packs)             │
│  Android packs · iOS packs · Windows packs · IoT packs   │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│                    Decoding Core                         │
│  SQLite · plist · protobuf · XML/EVTX · hex · media      │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│              Evidence Store & Custody                    │
│  Case DB · Object storage for images · Audit log · Hash  │
└───────────────────────────┬─────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────┐
│           Ingest / Acquisition Adapters                  │
│  File image · ADB/backup · Cloud export · PCAP · eMMC    │
└─────────────────────────────────────────────────────────┘
```

**Khuyến nghị thiết kế:** parser viết dưới dạng plugin có:

- `supported_artifacts[]`
- `min/max app_version` hoặc OS version
- `parse(evidence_ctx) -> list[NormalizedFact]`
- `tests/` gắn với dataset lab (Digital Corpora / image trong case study)

---

## 7. Roadmap sản phẩm đề xuất

### Giai đoạn A — MVP (3–6 tháng tùy đội)

- Case + evidence import + hash custody
- File browser + global search
- Android logical: app list + SMS/Call/Contacts + SQLite explorer
- Timeline thủ công (tag sự kiện)
- Report PDF cơ bản

**Tiêu chí thành công:** điều tra viên hoàn thành một case Android mẫu end-to-end và xuất báo cáo có provenance.

### Giai đoạn B — Mobile depth

- iOS native packs (Messages, Safari, Photos, Location)
- Conversation UI + map timeline
- WhatsApp (hoặc 1–2 app ưu tiên thị trường)
- Deleted record recovery cho SQLite
- Export JSON theo schema CFO/STIX-subset

### Giai đoạn C — Platform

- Windows disk artifact packs (nếu dual-offer computer+mobile)
- Plugin SDK cho đối tác/parser community
- IoT image modules
- Multi-user + audit enterprise

### Giai đoạn D — Intelligence

- Knowledge graph UI
- LLM-assisted entity/timeline drafting
- Cross-case search (cùng actor/device xuất hiện ở nhiều case)

---

## 8. Ma trận ưu tiên tính năng (tóm tắt điều hành)

| Tính năng | Digital | Mobile | Độ khó | Giá trị điều tra | Giai đoạn |
|-----------|:-------:|:------:|:------:|:----------------:|:---------:|
| Case & custody hash | ★ | ★ | Thấp | Rất cao | A |
| FS browser + search | ★ | ★ | Trung bình | Rất cao | A |
| SQLite explorer | ○ | ★ | Trung bình | Rất cao | A |
| Android comms parsers | ○ | ★ | Trung bình | Rất cao | A |
| Timeline đa nguồn | ★ | ★ | Cao | Rất cao | A/B |
| File carving | ★ | ○ | Cao | Cao | B |
| iOS KnowledgeC/Location | ○ | ★ | Cao | Cao | B |
| Third-party chat packs | ○ | ★ | Cao | Cao | B |
| STIX/CFO export | ★ | ★ | Trung bình | Trung bình–cao | B |
| Windows MFT/USN/Registry | ★ | ○ | Cao | Cao | C |
| IoT chip-off analysis | ○ | ★ | Cao | Trung bình–cao | C |
| AI knowledge graph | ★ | ★ | Rất cao | Cao (hỗ trợ) | D |

---

## 9. Nguồn học / kiểm thử trong repository này

Dùng làm **backlog nghiên cứu** và **bộ test thực tế**:

| Nhu cầu sản phẩm | Tham chiếu trong repo |
|------------------|------------------------|
| Quy trình & tool nền | `Basic_Computer_Skills_for_Forensics/`, `Help/` |
| Disk case lớn | `NIST_Data_Leakage_Case/`, `P2P_Leakage/` |
| Mobile Android | `Android10/` |
| Mobile iOS | `iOS/` |
| IoT / chip-off | `Echo_Device/`, `Eufy_Doorbell/`, `Drone_DJI_Mavic_Air/` |
| Mô hình evidence chuẩn hóa | `STIX_for_digital_forensics/` |
| AI / graph / profiling | `AI4Forensics/` |
| Paper học thuật | `papers/PST2022.pdf`, `papers/CIKM2024.pdf` |

**Gợi ý QA:** mỗi parser pack phải có ít nhất một fixture lấy từ lab/dataset công khai và một checklist “expected findings”.

---

## 10. Rủi ro & nguyên tắc tuân thủ khi làm sản phẩm

1. **Pháp lý & quyền riêng tư:** chỉ xử lý dữ liệu có thẩm quyền; tách môi trường lab và production.
2. **Công bố năng lực trung thực:** không tuyên bố “giống Cellebrite” nếu mới phủ subset artifact.
3. **Version drift:** app mobile đổi schema liên tục → cần CI theo dõi parser breakage.
4. **False confidence từ AI:** luôn gắn mức tin cậy và nguồn evidence gốc.
5. **Chain of custody kém** sẽ làm mất giá trị pháp lý dù parser rất mạnh.

---

## 11. Kết luận phương hướng

Để ra sản phẩm digital/mobile forensic đúng hướng:

1. **Đặt Case Workbench + Custody làm xương sống**, không bắt đầu bằng một parser rời.
2. **Chuẩn hóa evidence model sớm** (CFO/STIX-like) để timeline/graph/report dùng chung.
3. **Mobile-first:** SQLite + app parser packs + conversation/location/media.
4. **Dùng lab/dataset trong repo này làm know-how và regression tests**, không kỳ vọng repo thay cho engine thương mại.
5. **AI là lớp hỗ trợ sau khi đã có parse/timeline vững**, không thay thế provenance.

---

## 12. Next steps đề xuất cho đội sản phẩm

- [x] Đào sâu PRD **Digital Disk Forensics** (ổ cứng / USB / thẻ nhớ / disk image) — xem bộ tài liệu bên dưới
- [ ] Chọn thị trường mục tiêu mobile: Android-only / Android+iOS / mobile+IoT
- [ ] Chốt schema Case–Image–Artifact–Action–Timeline (v1) dùng chung disk + mobile sau này
- [ ] Lập danh mục parser pack P0 từ lab `Android10/` và `iOS/` (bước sau)
- [ ] Dựng MVP workbench disk theo [`digital-forensics-product/08_MVP_Roadmap_and_Backlog.md`](digital-forensics-product/08_MVP_Roadmap_and_Backlog.md)
- [ ] Gắn dataset lab vào pipeline kiểm thử tự động

---

## 13. Bộ PRD Digital Disk Forensics (đã tách riêng)

Phần sản phẩm **trích xuất / phục hồi từ ổ cứng, thẻ nhớ, USB và disk image** được viết chi tiết tại:

**[`docs/digital-forensics-product/00_README.md`](digital-forensics-product/00_README.md)**

| File | Nội dung |
|------|----------|
| [01_Product_Overview](digital-forensics-product/01_Product_Overview.md) | Tầm nhìn, persona, phạm vi |
| [02_Acquisition_and_Imaging](digital-forensics-product/02_Acquisition_and_Imaging.md) | Imaging, write-blocker, DD/E01 |
| [03_Filesystem_and_Deleted_Recovery](digital-forensics-product/03_Filesystem_and_Deleted_Recovery.md) | NTFS/exFAT, phục hồi metadata |
| [04_File_Carving_and_Signatures](digital-forensics-product/04_File_Carving_and_Signatures.md) | Carving & signature packs |
| [05_Search_Timeline_and_Analysis](digital-forensics-product/05_Search_Timeline_and_Analysis.md) | Search, tag, timeline |
| [06_Custody_Audit_and_Reporting](digital-forensics-product/06_Custody_Audit_and_Reporting.md) | Custody, audit, báo cáo |
| [07_Data_Model_and_Architecture](digital-forensics-product/07_Data_Model_and_Architecture.md) | Schema & kiến trúc |
| [08_MVP_Roadmap_and_Backlog](digital-forensics-product/08_MVP_Roadmap_and_Backlog.md) | Roadmap MVP |

**Mobile Forensic** vẫn chỉ ở mức định hướng trong tài liệu này; PRD chi tiết mobile sẽ làm ở bước sau.

---

*Tài liệu này tổng hợp và chuyển hóa nội dung giảng dạy/case study của repository Digital Forensics Lab thành góc nhìn định hướng sản phẩm. Cập nhật khi bổ sung lab mới hoặc thay đổi phạm vi thị trường.*
