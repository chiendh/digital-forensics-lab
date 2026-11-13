# 05 — iOS Artifact Packs

Dựa trên lab `iOS/` (iPhone / iOS 13.4.1, Digital Corpora). Chi tiết path/DB chủ yếu trong PPT; catalog dưới đây là **yêu cầu sản phẩm + thứ tự nghiên cứu**. Bổ sung path cụ thể = backlog R-03.

---

## 1. Lộ trình lab → pack

| Lab | Pack | Giá trị điều tra |
|-----|------|------------------|
| 0–2 | Device / image onboarding | Dossier |
| 3 | System settings | Accounts, locale |
| 4 | App lifecycle | Hiểu container |
| 5 | Messages | SMS/iMessage threads |
| 6 | Contacts | Address book |
| 7 | Calendar | Events |
| 8 | Safari | History/visits |
| 9 | Photos | Library + metadata |
| 10 | **KnowledgeC** | Behavioral / app usage timeline |
| 11 | Health | Sensor/health samples |
| 12 | Location | Movement narrative |
| 13–14 | Cellebrite / AXIOM | **Eval only** |
| 15 | Jailbreak | Trust/scope |

---

## 2. Pack P0

### 2.1. Communications & PII

- Messages: conversation UI, attachments ref, timestamps UTC  
- Contacts: identity graph nodes  
- Calendar: event facts → timeline  

### 2.2. Browser & media

- Safari: URL, visit time, title nếu có  
- Photos: gallery, EXIF GPS nếu có → map  

### 2.3. Device dossier

- Model, iOS version, identifiers readable from dump  

---

## 3. Pack P1 — Differentiators

| Pack | Vì sao quan trọng |
|------|-------------------|
| **KnowledgeC** | Lab nhấn hành vi dùng thiết bị — khác biệt so với chỉ đọc tin nhắn |
| **Health** | Bối cảnh thời gian/hoạt động vật lý |
| **Location** | Timeline di chuyển; correlate Photos/Messages |

UI: behavioral timeline riêng (màu/nguồn `knowledgeC` / `health` / `location`).

---

## 4. Tool parity labs (13–14) — không phải feature clone

Sản phẩm dùng lab này để:

1. Chọn tập **expected artifacts** trên cùng image.  
2. Điền gap matrix (App2 vs Cellebrite vs AXIOM vs thủ công).  
3. Viết mục Limitations trong báo cáo học thuật / user docs.  

Không đặt mục tiêu MVP “feature parity”.

---

## 5. Yêu cầu

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| IO-01 | Import iOS dump/backup structure | P0 |
| IO-02 | Messages+Contacts+Calendar packs | P0 |
| IO-03 | Safari+Photos packs | P0 |
| IO-04 | KnowledgeC timeline events | P1 |
| IO-05 | Health + Location packs | P1 |
| IO-06 | Jailbreak badge + scope notes | P0 |
| IO-07 | Gap matrix template filled once | P0 academic |

---

## 6. Acceptance

- Thread Messages hiển thị trên corpora iOS 13.  
- Safari visit list exportable.  
- KnowledgeC: ≥ N usage events trên fixture khi pack P1 xong.  
- Tài liệu gap matrix tồn tại trong `docs/academic` hoặc appendix report.  
