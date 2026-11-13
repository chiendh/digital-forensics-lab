# Digital Forensics Product Docs — Disk / USB / Memory Card / Disk Image

> Bộ tài liệu định hướng sản phẩm **Digital Forensic** tập trung **trích xuất và phục hồi dữ liệu** từ ổ cứng, thẻ nhớ, USB và file disk image.
>
> **Đối tượng:** cơ quan điều tra / giám định tư pháp.
> **Nền tảng MVP:** Windows desktop.
> **Ngoài phạm vi bộ này:** Mobile Forensic chi tiết (xem tài liệu tổng quát; sẽ đào sâu sau).

---

## Mục lục bộ tài liệu

| File | Nội dung |
|------|----------|
| [01_Product_Overview.md](01_Product_Overview.md) | Tầm nhìn, persona, giá trị, phạm vi in/out |
| [02_Acquisition_and_Imaging.md](02_Acquisition_and_Imaging.md) | Thu thập, imaging, write-blocker, định dạng image |
| [03_Filesystem_and_Deleted_Recovery.md](03_Filesystem_and_Deleted_Recovery.md) | Partition, NTFS/exFAT, phục hồi qua metadata |
| [04_File_Carving_and_Signatures.md](04_File_Carving_and_Signatures.md) | File signature, carving, preview, độ tin cậy |
| [05_Search_Timeline_and_Analysis.md](05_Search_Timeline_and_Analysis.md) | Search, tag, timeline, workflow phân tích |
| [06_Custody_Audit_and_Reporting.md](06_Custody_Audit_and_Reporting.md) | Chain of custody, audit, báo cáo giám định |
| [07_Data_Model_and_Architecture.md](07_Data_Model_and_Architecture.md) | Schema Case DB, kiến trúc module, API nội bộ |
| [08_MVP_Roadmap_and_Backlog.md](08_MVP_Roadmap_and_Backlog.md) | Roadmap giai đoạn A/B, epic, tiêu chí chấp nhận |

## Tài liệu liên quan trong repo

- Tổng quan digital + mobile: [`../Product_KnowHow_Digital_Mobile_Forensics.md`](../Product_KnowHow_Digital_Mobile_Forensics.md)
- Lab nền tảng (carving, Sleuth Kit, USB acquisition): `Basic_Computer_Skills_for_Forensics/`
- Case disk lớn: `NIST_Data_Leakage_Case/`, `P2P_Leakage/`
- Mô hình evidence chuẩn hóa: `STIX_for_digital_forensics/`
- Cài tool tham chiếu: `Help/`, mục Tools trong `README.md`

## Nguyên tắc đọc nhanh

1. Đọc **01** để thống nhất phạm vi sản phẩm.
2. Đọc **02 → 04** nếu bạn làm engine acquisition / FS / carving.
3. Đọc **05 → 06** nếu bạn làm UX điều tra / báo cáo pháp lý.
4. Đọc **07 → 08** nếu bạn lập kế hoạch kỹ thuật và sprint.

## Trạng thái

| Mục | Trạng thái |
|-----|------------|
| Phạm vi Digital Disk Forensics | Đã chốt (2026-11-13) |
| Mobile Forensic chi tiết | Hoãn — bước sau |
| Implementation plan / code | Chưa — chỉ tài liệu sản phẩm |
