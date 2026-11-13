# 01 — Product Overview: Mobile Forensics Workbench

## 1. Tóm tắt

**Mobile Forensics Workbench** là ứng dụng phân tích / phục hồi dữ liệu từ:

- Ảnh/dump **Android** (logical, filesystem, physical khi có)
- Ảnh/dump **iOS** (backup/logical/jailbreak image khi có)
- Ảnh **IoT companion** thu từ chip-off / microSD (Echo, Eufy, drone controller+SD)

Mục tiêu kép:

1. **Học thuật / đánh giá:** đo coverage artifact, PreDIE, gap so với tool thương mại.  
2. **Sản phẩm giám định:** case custody + parser packs + timeline + báo cáo.

Chia sẻ ontology và report engine với **App Disk**; khác lớp acquisition & parsers.

---

## 2. Vấn đề

- Evidence mobile nằm trong **SQLite / plist / protobuf / XML**, schema đổi theo version app.  
- Thu thập bị giới hạn mã hóa, SE/TEE, root/jailbreak policy.  
- Lab dạy walkthrough thủ công; giám định viên cần **workbench** tái lập được.  
- IoT (chuông cửa, loa thông minh, drone) ngày càng xuất hiện cùng điện thoại.

---

## 3. Persona

| Persona | Nhu cầu |
|---------|---------|
| Giám định viên mobile | Thread tin nhắn, call, location, media, báo cáo có provenance |
| Nghiên cứu viên / giảng viên | Fixture Digital Corpora; rubric đánh giá parser |
| Kỹ thuật viên chip-off | Ingest eMMC image, partition map, không cần UI “tháo chip” |

---

## 4. Phạm vi

### In scope

- Import dump/image có sẵn (MVP) + ghi nhận acquisition metadata  
- Device dossier (model, OS, identifiers khi đọc được)  
- App inventory + SQLite generic explorer  
- Parser packs: Android AOSP/GMS cốt lõi; iOS native cốt lõi  
- Maps/places decode mẫu (`gmm_myplaces`)  
- IoT: ingest Echo/Eufy-style trees + carving reuse từ App Disk  
- Timeline / tag / finding / PDF+JSON report / PreDIE gates  
- Capability & trust badges (logical vs rooted vs jailbreak vs chip-off)

### Out of scope (MVP)

- Bypass mã hóa trái pháp luật / exploit lock screen  
- Đầy đủ decoder WhatsApp mọi version (chỉ starter pack + giới hạn công bố)  
- Cloud live API (iCloud/Google) — chỉ import export người dùng cung cấp hợp pháp  
- Thay thế Cellebrite/AXIOM về bề rộng acquisition  

---

## 5. Nguyên tắc

1. Cùng **không ghi đè nguồn** + hash + audit như App Disk.  
2. Mỗi artifact row → file DB → image → case.  
3. Parser **versioned**; fail mềm khi schema lạ.  
4. Root/jailbreak/chip-off = **mở rộng phạm vi**, phải gắn cảnh báo độ tin cậy/pháp lý quy trình.  
5. So sánh commercial tools chỉ trong **eval matrix**, không marketing parity giả.

---

## 6. Happy path

```text
Tạo Case → Đăng ký Mobile/IoT Evidence
  → Import dump/image + hash
  → Device dossier + partition/sandbox tree
  → App inventory
  → Chạy parser packs (native → GMS/iOS advanced → optional 3rd party)
  → (Optional) Carve unallocated / media
  → Tag + Findings + Timeline
  → Official report nếu PreDIE pass
```

---

## 7. Quan hệ với lab repo

| Lab | Đóng góp trực tiếp |
|-----|-------------------|
| `Android10/` | Thứ tự điều tra + Maps script + SQLite pedagogy |
| `iOS/` | Pack list + KnowledgeC/Health/Location + tool parity labs |
| `Echo_Device/lab_data` | IoT fixture thật (XML, DB, carved JPG) |
| `Eufy_Doorbell/` | Quy trình P2P/log/camera/SQLite |
| `Drone_DJI_Mavic_Air/` | Mobile controller + microSD correlation |
| Academic docs | Eval + STIX/PreDIE |

---

## 8. Success metrics (định hướng)

- Hoàn thành case Android Corpora mẫu end-to-end với ≥1 conversation view + report  
- Decode đúng places từ `gmm_myplaces` fixture  
- Echo: đọc `build.prop` + Wi-Fi config + liệt kê carved JPG từ lab_data  
- Gap matrix iOS điền được sau 1 vòng so với tài liệu lab 13–14  
