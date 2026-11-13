# 04 — Presentable Evidence, Knowledge Graph & STIX/CFO

Tài liệu học thuật dùng chung cho **App Disk** và **App Mobile**, dựa trên:

- `STIX_for_digital_forensics/` (xSTIX / Cyber Forensic Objects)  
- Paper PST 2022 — *Visualizing and Reasoning about Presentable Digital Forensic Evidence with Knowledge Graphs* (`papers/PST2022.pdf`)  
- COMPSAC 2022 — shared instructional materials / artifact concepts (`papers/compsac2022.pdf`)  
- CIKM 2024 — LLM → entity/relation → STIX (`papers/CIKM2024.pdf`, `AI4Forensics/`)

---

## 1. Artifact vs Evidence

| Khái niệm | Nghĩa trong lab/repo | Hệ quả sản phẩm |
|-----------|----------------------|-----------------|
| **Artifact** | Dữ liệu kỹ thuật (file, row SQLite, log) | Object parse được |
| **Evidence (CFI)** | Artifact đã gắn ngữ cảnh điều tra, tool, provenance | Finding confirmed |
| **Action** | Hành vi ACID có ý nghĩa (copy USB, gửi tin, mở app) | `TimelineEvent` / `x-action` |
| **Crime case** | Bối cảnh vụ việc | `Case` |

Sản phẩm phải giúp chuyển Artifact → Evidence có kiểm chứng, không dừng ở viewer SQL.

---

## 2. PreDIE — năm tiêu chí bằng chứng trình bày được

| # | Tiêu chí | Định nghĩa ngắn | Gate trong app |
|---|----------|-----------------|----------------|
| 1 | **Authenticity** | Truy vết nguồn gốc (image, path, offset) | Bắt buộc provenance |
| 2 | **Integrity** | Không bị sửa; hash khớp | verify trước official |
| 3 | **Validity** | Sinh ra bởi phương pháp/tool đã biết phiên bản | engine_versions |
| 4 | **Credibility** | Người chịu trách nhiệm xác nhận | confirmed_by |
| 5 | **Relevance** | Liên hệ logic với case/action | link finding↔timeline↔case |

**Reasoning gợi ý từ PST2022:**

- Relevance ≈ tồn tại đường đi trên đồ thị từ artifact tới case  
- Authenticity ≈ đường đi chỉ gồm quan hệ chứa đựng / extract-from  
- Validity ≈ có node tool với input/output refs  

---

## 3. Ontology tối thiểu (product v1 = CFO subset)

```text
x-crime-case          → Case
x-investigator        → User/Examiner
x-image               → ImageAsset
x-disk-partition      → Partition (Disk; cũng dùng cho GPT trên eMMC)
x-secondary-Storage   → EvidenceSource disk/USB/eMMC/SD
x-action              → Finding narrative / TimelineEvent (verb)
x-timeline            → Timeline (ordered actions)
x-investigation-tool  → Engine module + version
file / artifact facts → FsEntry, CarvedObject, MobileArtifact, ExtractedFile
indicator+observed-data → optional khi export STIX đầy đủ
```

### Quan hệ nên hỗ trợ

`evidence-of`, `image-of`, `part-of`, `contains`, `extracted-from`, `indicated-by`, `based-on`, `performed-by`, `investigates`, `reconstructed-from`, `acquired-using`

### Động từ hành động (`x-activity-name-ov` — chọn lọc)

`acquire`, `hash`, `verify`, `mount`, `parse`, `extract`, `carve`, `recover`, `search`, `decode`, `timeline`, `report`, `confirm`

Dùng cho audit log + verb của action.

---

## 4. Mobile-specific facts (mở rộng ontology)

Không tách Case model; thêm **fact types**:

| Fact type | Ví dụ nguồn lab |
|-----------|-----------------|
| `message` / `call` | Android AOSP/GMS, iOS Messages |
| `contact` | contacts EAV / contacts DB |
| `place` | `gmm_myplaces.db` sync_item |
| `app_usage` | iOS KnowledgeC |
| `health_sample` | iOS Health |
| `location_point` | iOS Location / Maps / drone GPS |
| `wifi_network` | Echo `WifiConfigStore.xml` |
| `account` | Echo `accounts_*.db` |
| `media_item` | Photos; Echo foremost JPG |

Mỗi fact vẫn phải có `extracted-from` → file/DB → image → case.

---

## 5. Digital Forensic Artifact Generator (DFAG) — gợi ý phân module

Từ hướng COMPSAC: tổ chức parser theo **nguồn sinh artifact**, không theo nút UI ngẫu nhiên:

| Generator | App1 | App2 |
|-----------|------|------|
| Device/OS | Windows volume, MFT | Android/iOS settings, build.prop |
| File system | NTFS/exFAT | ext4/userdata, AFC dumps |
| Application | Browser/email (B) | SMS, Maps, WhatsApp… |
| Network/IoT | Optional PCAP | Echo/Eufy P2P, Wi-Fi, drone |
| Human analyst | Manual timeline tags | Manual tags |

Coverage matrix công bố theo DFAG = tài liệu học thuật tốt.

---

## 6. Export STIX 2.1 + x-* 

### MVP học thuật / sản phẩm giai đoạn B

- Export JSON bundle: Case, Image, File(s), Relationship evidence-of / extracted-from  
- Custom objects `x-image`, `x-timeline` theo convention `x-<vendor>-…` nếu cần  

### Golden samples trong repo

- `STIX_for_digital_forensics/Illegal_Possession_Images/`  
- `STIX_for_digital_forensics/Email_Harassment/`  

Dùng để so **shape** export (không cần byte-identical).

### AI path (CIKM)

`narrative/log → entities → relations → STIX draft → human confirm → bundle`  
Zero-shot vs one-shot phải đo trong evaluation ([03](03_Academic_Evaluation_Framework.md)).

---

## 7. Câu narrative mẫu (báo cáo)

> Action **Copy** (`x-action`) được **indicated-by** sự hiện diện file F trên USB (FsEntry) **extracted-from** Image I (`x-image`, SHA-256=…) **acquired-using** Tool T v0.1 **investigated-by** Examiner E trong Case C.

Tiếng Việt giám định:

> File F được trích xuất từ ảnh đĩa I (SHA-256 …) bằng công cụ T phiên bản …; thời gian hệ thống tệp …; liên hệ với hành vi … trong vụ C. Người xác nhận: …

---

## 8. Acceptance học thuật cho lớp KG/STIX

- [ ] Mọi Finding confirmed có path tới Image + Case  
- [ ] Mọi official report ghi tool versions  
- [ ] Export JSON mở được và validate field bắt buộc  
- [ ] (B) Graph UI chỉ ra được cạnh Authenticity / Relevance  
- [ ] (C) AI draft không vào official khi chưa confirm  

---

## 9. Tham chiếu trích dẫn

Khi viết paper/tech report dựa trên framework này, cite:

- Xu & Xu, PST 2022 — Presentable evidence + knowledge graphs  
- COMPSAC 2022 — Shared digital forensics instructional materials  
- Xu et al., CIKM 2024 — LLMs for digital forensics  
- Repo: Frank Xu et al., *Digital Forensics Lab* (và fork đang dùng)
