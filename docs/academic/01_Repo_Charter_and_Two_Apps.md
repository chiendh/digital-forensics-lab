# 01 — Repo Charter & Two Recovery Applications

## 1. Hiến chương (Charter)

### 1.1. Sứ mệnh của repository

Repository **Digital Forensics Lab** (fork/nguồn học thuật) phục vụ:

| Vai trò | Mô tả |
|---------|--------|
| **Nghiên cứu & giảng dạy** | Lab tương tác, case NIST/Digital Corpora, bài tập artifact |
| **Tài liệu học thuật** | Paper, STIX/CFO, AI4Forensics, template báo cáo |
| **Đánh giá kỹ thuật** | Ground truth / fixture để đo parser, carving, timeline, report |
| **Đầu vào R&D sản phẩm** | Know-how → PRD cho **hai ứng dụng** trích xuất & phục hồi |

### 1.2. Những gì repo **không** phải

- Không phải SDK thương mại hay binary acquisition như Cellebrite/AXIOM  
- Không thay thế quy trình tố tụng / chứng nhận pháp y  
- Không chứa (đầy đủ) dump mobile/iOS trong-tree; nhiều lab trỏ Digital Corpora / link ngoài  
- Không phải nơi commit mã nguồn kín của hai ứng dụng (có thể repo code riêng)

### 1.3. Nguyên tắc sử dụng cho đội sản phẩm

1. **Lab = curriculum + fixture**, không copy slide thành UI.  
2. Mọi finding trong app phải vượt tiêu chí **PreDIE** (xem [04](04_Evidence_Knowledge_Graph_and_STIX.md)).  
3. So sánh với tool thương mại chỉ để **đo gap**, không tuyên bố tương đương khi chưa đủ coverage.  
4. AI (nếu có) chỉ **đề xuất**; người xác nhận.  
5. Mobile và Disk **chia sẻ ontology Case–Image–Fact–Action–Timeline**; khác ở lớp ingest & parser pack.

---

## 2. Hai ứng dụng mục tiêu

```text
                    ┌─────────────────────────────────────┐
                    │   Academic Lab Repository (this)    │
                    │  know-how · fixtures · STIX · eval  │
                    └───────────────┬─────────────────────┘
                                    │
                 ┌──────────────────┴──────────────────┐
                 ▼                                     ▼
┌────────────────────────────┐       ┌────────────────────────────┐
│ App 1: Disk Recovery       │       │ App 2: Mobile Recovery     │
│ HDD/SSD · USB · SD · Image │       │ Android · iOS · IoT dumps  │
│ docs/digital-forensics-…   │       │ docs/mobile-forensics-…    │
└────────────────────────────┘       └────────────────────────────┘
```

### 2.1. App 1 — Digital Disk Forensics Workbench

| Khía cạnh | Nội dung |
|-----------|----------|
| **Vấn đề** | Trích xuất / phục hồi từ ổ cứng, thẻ nhớ, USB, disk image có custody |
| **Người dùng** | Giám định tư pháp / lab điều tra disk |
| **Năng lực lõi** | Imaging, FS parse (NTFS/exFAT), deleted recovery, carving, search, report |
| **PRD** | [`../digital-forensics-product/`](../digital-forensics-product/00_README.md) |
| **Lab neo** | USB acquisition, Sleuth Kit, Data Carving, NIST leakage, P2P, Rhino |

### 2.2. App 2 — Mobile / Companion-Device Forensics Workbench

| Khía cạnh | Nội dung |
|-----------|----------|
| **Vấn đề** | Phân tích & phục hồi artifact từ điện thoại / backup / chip-off IoT liên quan mobile |
| **Người dùng** | Giám định mobile; lab nghiên cứu artifact app |
| **Năng lực lõi** | Import logical/FS/chip-off image; SQLite packs; conversation/location/media; timeline; report |
| **PRD** | [`../mobile-forensics-product/`](../mobile-forensics-product/00_README.md) |
| **Lab neo** | `Android10/`, `iOS/`, `Echo_Device/`, `Eufy_Doorbell/`, `Drone_DJI_Mavic_Air/` |

### 2.3. Phần dùng chung (nên một thư viện / schema)

- Case / Investigator / Audit / Custody  
- ImageAsset + hash verify  
- Finding / Tag / TimelineEvent / Report export  
- PreDIE gates + (sau này) STIX/CFO export  
- Job runner, hex/text/image preview  

Khác biệt chủ yếu: **Acquisition adapters** và **Artifact parser packs**.

---

## 3. Chuỗi giá trị từ học thuật → sản phẩm

```text
Lab PPT + dataset
  → Artifact catalog & investigation order
  → Evaluation rubric + expected findings
  → PRD module (Disk / Mobile)
  → Engine + Workbench implementation (repo riêng)
  → Regression trên fixture lab / Digital Corpora
  → Academic write-up (coverage, PreDIE, tool gap)
```

---

## 4. Định nghĩa “hoàn thành tài liệu nghiên cứu” cho repo docs

Đối với mục tiêu *phục vụ đánh giá + tài liệu học thuật + định hướng 2 app*, bộ docs được coi là đủ khi có:

1. Charter + mapping lab→app (file này + [02](02_Lab_to_Product_Mapping.md))  
2. Khung đánh giá học thuật ([03](03_Academic_Evaluation_Framework.md))  
3. Mô hình evidence/KG/STIX ([04](04_Evidence_Knowledge_Graph_and_STIX.md))  
4. PRD Disk đủ module (đã có)  
5. PRD Mobile đủ module (bộ `mobile-forensics-product/`)  
6. Backlog nghiên cứu còn mở ([05](05_Shared_Research_Backlog.md))  

Phần **code sản phẩm** nằm ngoài định nghĩa hoàn thành của charter này.
