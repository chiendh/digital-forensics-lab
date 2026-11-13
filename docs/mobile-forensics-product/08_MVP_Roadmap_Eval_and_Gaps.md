# 08 — Mobile MVP Roadmap, Evaluation & Gaps

## 1. Giai đoạn M-A — MVP Mobile Workbench

| Epic | Deliverable |
|------|-------------|
| M1 | Case reuse + import dump/image + hash + badges |
| M2 | Android tree inventory + SQLite explorer |
| M3 | AOSP messages/contacts (+ calendar nếu kịp) |
| M4 | Maps `gmm_myplaces` pack (= script parity) |
| M5 | Call log + basic search/tag/finding |
| M6 | Timeline nhẹ + PDF/JSON report + PreDIE |
| M7 | Echo ingest dossier + Wi-Fi + reuse carving |

**DoD M-A:** Case Android Corpora-style end-to-end + places pack đúng + report official.

---

## 2. Giai đoạn M-B — iOS & depth

| Epic | Nội dung |
|------|----------|
| M8 | iOS messages/contacts/safari/photos |
| M9 | KnowledgeC + Location (+ Health) |
| M10 | WhatsApp/Kik/TextNow best-effort + detection |
| M11 | SQLite deleted recovery |
| M12 | Gap matrix vs Cellebrite/AXIOM trên 1 image |
| M13 | Eufy presets + drone GPS gallery |

---

## 3. Giai đoạn M-C — Platform

- Live acquisition adapters  
- Plugin SDK parser packs  
- Multi-device graph (CTF/group crime)  
- Cloud export ingest  
- AI assist (sau khi L1 parsers ổn) — theo academic AI eval  

---

## 4. Evaluation gắn lab

Chi tiết metric: [`../academic/03_Academic_Evaluation_Framework.md`](../academic/03_Academic_Evaluation_Framework.md)

| Fixture | Kiểm gì ở MVP |
|---------|----------------|
| Android 10 corpora | M3–M6 |
| `gmm_myplaces.db` | M4 parity |
| Echo `lab_data` | M7 |
| iOS 13 corpora | M8+ |

---

## 5. Gap matrix template (điền khi đánh giá)

| Hạng mục | App2 MVP | Manual lab OSS | Cellebrite | AXIOM |
|----------|----------|----------------|------------|-------|
| Logical import | | | | |
| Physical/chip-off ingest | | | | |
| SMS/iMessage | | | | |
| WhatsApp depth | | | | |
| KnowledgeC | | | | |
| Location | | | | |
| IoT Echo/Eufy | | | | |
| Deleted SQLite | | | | |
| Custody/report PreDIE | | | | |

---

## 6. Rủi ro

| Rủi ro | Ứng phó |
|--------|---------|
| Schema drift app | Pack version + fail mềm + CI fixture |
| PPT không có path plaintext | R-02/R-03 corpora catalog |
| Kỳ vọng parity thương mại | Charter + gap matrix công khai |
| Encryption | Công bố giới hạn; không bypass bất hợp pháp |
| Image lớn IoT | Worker process; hash progress |

---

## 7. User stories mẫu

1. Import Android dump và xem danh sách package.  
2. Đọc SMS threads và gắn finding.  
3. Xuất places từ Maps DB ra map + CSV.  
4. Mở Echo userdata, xem Wi-Fi đã lưu, index ảnh carved.  
5. Xuất báo cáo có acquisition badge và hash.  

---

## 8. Thứ tự đọc khi triển khai

1. Academic charter 01–02  
2. Mobile 01–03  
3. Android 04 (+ Maps first)  
4. Analysis/custody 07  
5. IoT 06 (Echo)  
6. iOS 05 (giai đoạn B)  
