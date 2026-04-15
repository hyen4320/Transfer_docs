---
tags: [project, transfermap]
status: "🟡 진행 중"
started: 2026-04-14
stack: [Spring Boot, PostgreSQL, Redis, React, D3.js]
---

# TransferMap

> 해외축구 이적시장 소식을 X(트위터) 기자 게시물로 수집하고,  
> 기자 공신력을 산출해 유럽 지도 UI에 제공하는 서비스.

---

## 빠른 이동

| 문서 | 링크 |
|------|------|
| 개발 계획 | [[dev-plan]] |
| 기술 스택 | [[004 - 기술 스택 선정]] |
| ERD | [[erd]] |
| 기자/구단 목록 | [[leagues-teams-journalists\|📂 Resources/leagues-teams-journalists]] |
| 칸반 보드 | [[📌 Kanban]] |

---

## 아키텍처

```
X API (기자 타임라인)
    ↓ @Scheduled 15분
POST 수집 → TRANSFER_NEWS 파싱
    ↓
PostgreSQL (PostGIS)  ←→  Redis 캐시
    ↓
Spring Boot REST API
    ↓
React + D3.js (유럽 지도)
```

**공신력 공식:** `credibility = speed × 0.3 + accuracy × 0.5 + impact × 0.2`

---

## 진행 현황

| 영역 | 상태 |
|------|------|
| BE - Domain / Repository | ✅ 완료 |
| BE - Service 비즈니스 로직 | 🔲 미구현 |
| BE - REST Controller | 🔲 미작성 |
| BE - X API 수집 파이프라인 | 🔲 미구현 |
| DB - Docker 환경 + 시딩 | 🔲 미완 |
| FE - React 앱 | 🔲 없음 |
| Infra - Docker / 배포 | 🔲 없음 |

---

## ADR (Architecture Decision Records)

```dataview
TABLE file.ctime AS "작성일"
FROM "01_Projects/TransferMap/ADR"
SORT file.ctime ASC
```

---

## 트러블슈팅 기록

```dataview
TABLE file.mtime AS "최근 수정"
FROM "01_Projects/TransferMap/Troubleshooting"
SORT file.mtime DESC
```

---

## 회의록

```dataview
TABLE file.ctime AS "날짜"
FROM "01_Projects/TransferMap/Meetings"
SORT file.ctime DESC
```
