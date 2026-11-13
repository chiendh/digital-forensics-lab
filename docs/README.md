# Documentation Index — Digital & Mobile Forensics Lab → Two Recovery Apps

Repository này phục vụ **nghiên cứu / phân tích học thuật** về digital & mobile forensics, và là nền tảng đánh giá + tài liệu để phát triển **hai ứng dụng trích xuất & phục hồi dữ liệu**.

```text
docs/
├── README.md                          ← bạn đang ở đây
├── Product_KnowHow_Digital_Mobile_Forensics.md
├── academic/                          ← hiến chương lab, mapping, eval, STIX/PreDIE
├── digital-forensics-product/         ← PRD App 1 (Disk / USB / SD / image)
└── mobile-forensics-product/          ← PRD App 2 (Android / iOS / IoT)
```

---

## Đọc theo vai trò

| Vai trò | Bắt đầu tại |
|---------|-------------|
| Nghiên cứu viên / giảng viên | [`academic/00_README.md`](academic/00_README.md) |
| Product / kỹ thuật App Disk | [`digital-forensics-product/00_README.md`](digital-forensics-product/00_README.md) |
| Product / kỹ thuật App Mobile | [`mobile-forensics-product/00_README.md`](mobile-forensics-product/00_README.md) |
| Muốn overview nhanh | [`Product_KnowHow_Digital_Mobile_Forensics.md`](Product_KnowHow_Digital_Mobile_Forensics.md) |

---

## Hai ứng dụng

| App | Phạm vi | PRD |
|-----|---------|-----|
| **1 — Disk Recovery Workbench** | Ổ cứng, USB, thẻ nhớ, disk image (DD/E01) | [`digital-forensics-product/`](digital-forensics-product/00_README.md) |
| **2 — Mobile Recovery Workbench** | Android, iOS, Echo/Eufy/drone dumps | [`mobile-forensics-product/`](mobile-forensics-product/00_README.md) |

Cùng Case/Custody/PreDIE/Report; khác Acquisition + Parser packs. Chi tiết: [`academic/01_Repo_Charter_and_Two_Apps.md`](academic/01_Repo_Charter_and_Two_Apps.md).

---

## Trạng thái tài liệu

| Hạng mục | Trạng thái |
|----------|------------|
| Know-how tổng quát | Có |
| PRD Disk 00–08 | Có |
| Academic charter, mapping, eval, STIX | Có |
| PRD Mobile 00–08 | Có |
| Artifact path catalog đầy đủ từ corpora | Starter + backlog R-02/R-03 |
| Gold `expected_findings` fixtures | Backlog R-01 |
| Mã nguồn hai ứng dụng | Ngoài phạm vi repo lab |
