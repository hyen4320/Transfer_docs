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

> 마지막 업데이트: 2026-04-30

### Backend

| 영역 | 상태 | 비고 |
|------|------|------|
| Domain / Entity | ✅ 완료 | Post, Journalist, Club, Player, League, TransferNews, CredibilityMetric, Verification |
| Repository | ✅ 완료 | 전 엔티티 Spring Data JPA Repository 구현 |
| Service — 공신력 계산 | ✅ 완료 | speed × 0.3 + accuracy × 0.5 + impact × 0.2 |
| Service — 이적 뉴스 / 기자 / 선수 | ✅ 완료 | TransferNewsServiceImpl, JournalistServiceImpl 등 |
| REST Controller | ✅ 완료 | News / Club / Journalist / Player / League + GlobalExceptionHandler |
| Caffeine 캐시 | ✅ 완료 | news:feed (TTL 10분), journalist:ranking (TTL 30분) |
| Flyway DB 마이그레이션 | ✅ 완료 | V8 — Journalist.isBot 컬럼 추가 |
| CORS 설정 | ✅ 완료 | localhost:5173 허용 |
| X API 수집 스케줄러 | 🟡 부분 완료 | 15분 주기 수집 + Redis rate-limit ✅ / POST→TransferNews 파싱 🔲 |
| Trending API | 🔲 미완 | `GET /api/news/trending-players` 미구현 |
| 루머 확정 로직 | 🔲 미완 | X 수집 초안만, DB 저장·상태 관리 미구현 |

### Frontend

| 영역 | 상태 | 비고 |
|------|------|------|
| 유럽 지도 (WorldMap) | ✅ 완료 | D3 geoMercator, 구단 마커 겹침 해소(resolveOverlaps) |
| WorldMap 터치 제스처 | ✅ 완료 | pinch zoom + pan (CSS transform), 더블탭 reset |
| WorldMap 성능 최적화 | ✅ 완료 | D3 setState 배칭(54ms→38ms), tooltipPos ref 전환(mousemove 렌더 20회 제거) |
| 사이드 패널 (SidePanel) | ✅ 완료 | 뉴스 피드 / 구단 상세 + 이적 탭 |
| 국가 리그 지도 (CountryMapPage) | ✅ 완료 | 구단 마커 클릭 → 사이드 패널 연동, 모바일 bottom sheet |
| 선수 패널 (PlayerPanel) | ✅ 완료 | 이적 히스토리 선 연동, 모바일 bottom sheet, 검색 toast |
| 기자 목록 / 상세 페이지 | ✅ 완료 | 공신력 점수 표시, 봇 제외 필터 |
| 선수 상세 페이지 | ✅ 완료 | 이적 히스토리 |
| 에러 페이지 (404 / 500) | ✅ 완료 | ErrorBoundary 포함 |
| BE↔FE 데이터 연동 | ✅ 완료 | apiFetch + mapper 레이어 (status/fee/time 변환), FreeAgent 폴백 |
| 모바일 반응형 | ✅ 완료 | useIsMobile 공용 훅, bottom sheet 패턴 전반 적용 |
| 성능 최적화 | ✅ 완료 | AbortController, React.memo(NewsCard), worldAtlas singleton 캐싱 |
| SEO / 메타태그 | ✅ 완료 | react-helmet-async, 선수·기자 페이지 동적 OG 태그 |
| 파비콘 | ✅ 완료 | 512×512 PNG, manifest.json 업데이트 |
| 시즌 연동 | ✅ 완료 | 뉴스피드 시즌 선택 → 메인 페이지 마커 동기화 |
| 상수 통합 | ✅ 완료 | LEAGUE_NAME_TO_ID, SEASON_OPTIONS → constants.ts 단일화 |
| Trending UI | 🔲 미완 | TRENDING 더미 상수 사용 중, API 연동 필요 |
| Canonical URL | 🔲 미완 | 각 페이지 `<link rel="canonical">` 미추가 |

### Infra / DB

| 영역 | 상태 | 비고 |
|------|------|------|
| Docker — PostgreSQL + Redis | ✅ 완료 | docker-compose.yml (PostGIS 16) |
| 5대 리그 DB 시딩 | ✅ 완료 | EPL, LaLiga, Bundesliga, SerieA, Ligue1 SQL 실행 |
| 클럽 좌표 수정 | ✅ 완료 | 35개 구단 fix_club_coordinates.sql 적용 |
| 기타 리그 데이터 | 🟡 부분 완료 | 네덜란드, 포르투갈 클럽 추가 (미완전) |
| 앱 배포 Docker 설정 | 🔲 미완 | BE / FE Dockerfile 미작성 |
| GitHub Actions CI/CD | 🔲 미완 | |
| Nginx 리버스 프록시 | 🔲 미완 | |
| 운영 배포 | 🔲 미완 | |
| Cloudflare WAF | 🔲 미완 | 계속 이월 중 |

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
