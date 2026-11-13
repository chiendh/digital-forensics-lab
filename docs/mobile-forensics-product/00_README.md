# Mobile Forensics Product Docs — Android / iOS / IoT Companion

> Bộ PRD định hướng **ứng dụng thứ hai**: trích xuất, phục hồi và phân tích dữ liệu **mobile forensic** (và thiết bị companion IoT liên quan), dựa trên lab học thuật trong repository.
>
> **Đối tượng nghiên cứu/sản phẩm:** giám định tư pháp + lab học thuật đánh giá artifact.  
> **Nền tảng MVP đề xuất:** Windows desktop (đồng bộ App Disk) hoặc cross-platform sau.  
> **Repo này:** nguồn know-how / fixture / đánh giá — không phải binary thương mại.

---

## Mục lục

| File | Nội dung |
|------|----------|
| [01_Product_Overview.md](01_Product_Overview.md) | Tầm nhìn, persona, in/out scope |
| [02_Acquisition_Modes_and_Trust.md](02_Acquisition_Modes_and_Trust.md) | Logical / FS / physical / chip-off / root / jailbreak |
| [03_SQLite_and_App_Inventory.md](03_SQLite_and_App_Inventory.md) | Sandbox, SQLite explorer, package catalog |
| [04_Android_Artifact_Packs.md](04_Android_Artifact_Packs.md) | AOSP / GMS / third-party |
| [05_iOS_Artifact_Packs.md](05_iOS_Artifact_Packs.md) | Messages → KnowledgeC / Health / Location |
| [06_IoT_Companion_Devices.md](06_IoT_Companion_Devices.md) | Echo, Eufy, DJI |
| [07_Analysis_Custody_and_Reporting.md](07_Analysis_Custody_and_Reporting.md) | Timeline, custody, report — tái sử dụng nền App Disk |
| [08_MVP_Roadmap_Eval_and_Gaps.md](08_MVP_Roadmap_Eval_and_Gaps.md) | Roadmap, eval, gap vs Cellebrite/AXIOM |

## Liên kết

- Academic charter: [`../academic/00_README.md`](../academic/00_README.md)
- App Disk PRD: [`../digital-forensics-product/00_README.md`](../digital-forensics-product/00_README.md)
- Know-how tổng: [`../Product_KnowHow_Digital_Mobile_Forensics.md`](../Product_KnowHow_Digital_Mobile_Forensics.md)
- Labs: `Android10/`, `iOS/`, `Echo_Device/`, `Eufy_Doorbell/`, `Drone_DJI_Mavic_Air/`
