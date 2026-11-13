# 03 — App Inventory & SQLite Workbench

## 1. Mục tiêu

Hầu hết artifact mobile lab (`Android10`, Echo SQLite, iOS DBs) đòi hỏi:

1. Liệt kê app / package / bundle  
2. Duyệt sandbox file  
3. Mở SQLite với schema discovery  
4. Query / export CSV  
5. Chuyển row → **typed MobileArtifact** khi có parser pack  

Module này là **nền** trước khi làm conversation UI.

---

## 2. Android app inventory

Từ lab scripts:

- Quét `data/data/` (hoặc tương đương trong dump) → `package_id`  
- Enrich tên app qua Play Store HTML hoặc bảng map nội bộ (offline ưu tiên cho lab air-gap)

```text
AppPackage
  package_id
  display_name
  version_name / version_code (nếu có)
  data_path
  has_sqlite_dbs[]
  parser_pack_id?   # nếu đã hỗ trợ
```

---

## 3. iOS inventory

- Bundle IDs dưới `AppDomain-*` / `HomeDomain` tùy định dạng dump  
- System DBs nằm ngoài app container (Messages, KnowledgeC, …) — đăng ký như **system packs**, không chỉ “installed apps”

---

## 4. SQLite explorer (P0)

| Năng lực | Chi tiết |
|----------|----------|
| Mở DB read-only | Copy-on-open vào `work/` nếu cần tránh lock |
| `.schema` / list tables | UI tree |
| Preview table | Phân trang |
| Export CSV | Có |
| BLOB inspector | hex + utf-8 try + length |
| Timestamp helper | Heuristic s/ms epoch → UTC |
| Free-page / WAL note | Cảnh báo; recovery sâu = P1 (undark-class) |

Pedagogy lab: `Android10/sqlite_tutorial` (CRUD, JOIN, **EAV contacts**) → UI nên hỗ trợ lưu query mẫu theo pack.

---

## 5. Từ generic SQL → typed artifact

Pipeline:

```text
DB file located
  → Pack matcher (path regex + table signature)
  → Parser version select
  → Normalized facts (message, call, place…)
  → Index for search/timeline
```

Nếu không match pack: vẫn cho explorer + “Mark as finding” thủ công.

---

## 6. Yêu cầu

| ID | Yêu cầu | Ưu tiên |
|----|---------|---------|
| SQ-01 | Inventory packages/bundles | P0 |
| SQ-02 | Tree sandbox + open file preview | P0 |
| SQ-03 | SQLite schema+table preview+CSV | P0 |
| SQ-04 | BLOB + timestamp helpers | P0 |
| SQ-05 | Query bookmarks per case | P1 |
| SQ-06 | Deleted row recovery (freelist/WAL) | P1 |
| SQ-07 | Offline package name DB | P1 |

---

## 7. Acceptance

- Trên dump Android mẫu: liệt kê được package; mở 1 DB; export CSV.  
- Chạy lại logic tương đương `5_2_3_extract_myplaces_sync_items.py` qua pack hoặc query bookmark.  
- EAV demo: query JOIN tái tạo contact từ teaching schema (học thuật).  
