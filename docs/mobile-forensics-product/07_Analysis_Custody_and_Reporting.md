# 07 — Analysis, Custody & Reporting (Mobile)

Tái sử dụng tối đa nền tảng đã mô tả cho App Disk; tài liệu này chỉ nêu **khác biệt mobile** và điểm neo học thuật.

Tham chiếu bắt buộc:

- Disk: `../digital-forensics-product/05_*.md`, `06_*.md`, `07_*.md`
- Academic: `../academic/03_Academic_Evaluation_Framework.md`, `04_Evidence_Knowledge_Graph_and_STIX.md`

---

## 1. Analysis UX đặc thù mobile

| View | Nội dung |
|------|----------|
| Conversation | Threads SMS/iMessage/app chat; bubble + attachment |
| Call log | Directional list → timeline |
| Map | Places (Maps), Photo EXIF, Location pack, drone GPS |
| Gallery | Photos + carved media |
| Behavioral | KnowledgeC / recent_tasks (Echo) |
| App inventory | Packages/bundles + pack status |

Search: metadata + message body + URL + phone number normalization.

---

## 2. Timeline sources (mobile)

| Nguồn | `ts_source` ví dụ |
|-------|-------------------|
| Message/call rows | `msg_date`, `call_date` |
| Safari/History | `visit_time` |
| KnowledgeC | `knowledgec_event` |
| Location | `location_sample` |
| IoT daily logs | `iot_daily_log` |
| Carved media | `carved_discovered` (không nhầm với thời điểm chụp) |
| Acquisition | `acquisition` |

Timezone: lưu UTC; hiển thị theo case timezone.

---

## 3. Custody — bổ sung field mobile/IoT

Ngoài sự kiện Disk, bắt buộc:

- `privilege_level_recorded`  
- `jailbreak_or_root_flag`  
- `destructive_acquisition` (chip-off)  
- `parser_pack_versions{}` khi chạy pack  
- `lab_dataset_citation` (Digital Corpora / VTO / …)

PreDIE gates giữ nguyên trước official report.

---

## 4. Report sections thêm cho mobile

1. Device dossier + acquisition mode/trust badges  
2. App inventory summary  
3. Communications findings  
4. Location/map findings  
5. IoT companion findings (nếu có)  
6. Limitations / encryption / unsupported schemas  
7. Gap note (optional academic appendix vs AXIOM/Cellebrite)  

JSON manifest: thêm `mobile_artifacts[]` với `fact_type`, `pack_id`, `db_path`, `row_ref`.

---

## 5. Shared code recommendation

```text
CaseStore, Audit, ReportEngine, CarveEngine, SearchIndex
    ↑ reused
MobileHost adds: AcquisitionAdaptersMobile, ParserPackRunner, ConversationUI, MapUI
```

Tránh fork schema Case giữa hai app.

---

## 6. Acceptance

- Finding tin nhắn confirmed có provenance db_path + row + image hash.  
- Official PDF có badge acquisition mode.  
- Timeline lọc theo `ts_source=knowledgec` khi pack có dữ liệu.  
