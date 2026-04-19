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

### Backend

| 영역 | 상태 | 비고 |
|------|------|------|
| Domain / Entity | ✅ 완료 | Post, Journalist, Club, Player, League, TransferNews, CredibilityMetric, Verification |
| Repository | ✅ 완료 | 전 엔티티 Spring Data JPA Repository 구현 |
| Service — 공신력 계산 | ✅ 완료 | speed × 0.3 + accuracy × 0.5 + impact × 0.2 |
| Service — 이적 뉴스 / 기자 / 선수 | ✅ 완료 | TransferNewsServiceImpl, JournalistServiceImpl 등 |
| REST Controller | ✅ 완료 | News / Club / Journalist / Player / League + GlobalExceptionHandler |
| X API 수집 스케줄러 | 🟡 부분 완료 | 15분 주기 수집 + Redis rate-limit ✅ / POST→TransferNews 파싱 🔲 |
| CORS 설정 | ✅ 완료 | localhost:5173 허용 |

### Frontend

| 영역 | 상태 | 비고 |
|------|------|------|
| 유럽 지도 (WorldMap) | ✅ 완료 | D3 geoMercator, 구단 마커 겹침 해소(resolveOverlaps) |
| 사이드 패널 (SidePanel) | ✅ 완료 | 뉴스 피드 / 구단 상세 + 이적 탭 |
| 국가 리그 지도 (CountryMapPage) | ✅ 완료 | 구단 마커 클릭 → 사이드 패널 연동 |
| 기자 목록 / 상세 페이지 | ✅ 완료 | 공신력 점수 표시 |
| 선수 상세 페이지 | ✅ 완료 | 이적 히스토리 |
| 에러 페이지 (404 / 500) | ✅ 완료 | ErrorBoundary 포함 |
| BE↔FE 데이터 연동 | ✅ 완료 | apiFetch + mapper 레이어 (status/fee/time 변환) |

### Infra / DB

| 영역                          | 상태    | 비고                              |
| --------------------------- | ----- | ------------------------------- |
| Docker — PostgreSQL + Redis | ✅ 완료  | docker-compose.yml (PostGIS 16) |
| DB 시딩                       | 🔲 미완 | 시드 스크립트 없음, 수동 입력 상태            |
| 앱 배포 Docker 설정              | 🔲 미완 | BE / FE Dockerfile 미작성          |
| 운영 배포                       | 🔲 미완 |                                 |

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
