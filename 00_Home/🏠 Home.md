---
tags: [home, dashboard]
---

# 🏠 Home

---

## 📌 진행 중인 프로젝트

```dataview
TABLE status AS "상태", file.mtime AS "최근 수정"
FROM "01_Projects"
WHERE contains(tags, "project")
SORT file.mtime DESC
```

---

## ✅ 미완료 태스크

```dataview
TASK
FROM "01_Projects"
WHERE !completed
SORT file.mtime DESC
LIMIT 20
```

---

## 📅 최근 데일리 노트

```dataview
LIST
FROM "02_Daily"
SORT file.name DESC
LIMIT 7
```

---

## 🔥 최근 수정 노트

```dataview
TABLE file.mtime AS "수정일"
WHERE file.folder != "Templates" AND file.name != "🏠 Home"
SORT file.mtime DESC
LIMIT 10
```
