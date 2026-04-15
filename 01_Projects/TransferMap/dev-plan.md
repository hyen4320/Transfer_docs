---
tags: [transfermap, planning]
type: plan
updated: 2026-04-14
---

# TransferMap 개발 계획서

## 현재 상태 요약

| 영역 | 완료 | 미완료 |
|------|------|--------|
| BE - Domain Model | ✅ 전체 (7개 엔티티) | — |
| BE - Repository | ✅ 전체 (6개) | — |
| BE - Service | ✅ 골격 생성 | ❌ 비즈니스 로직 미구현 |
| BE - Controller | ❌ 없음 | ❌ REST API 미작성 |
| BE - X API | ✅ 클라이언트 구조 | ❌ 수집 로직 미구현 |
| DB - 기본 데이터 | ✅ SQL 스크립트 (5대 리그) | ❌ DB 적용 미완 |
| DB - 선수 데이터 | ✅ 크롤러 (fd/tm) | ❌ 실행 미완 |
| FE - 디자인 시안 | ✅ 메인 페이지 HTML | ❌ 실제 앱 없음 |
| Infra | ❌ 없음 | ❌ Docker/배포 미설정 |

---

## Phase 1 — 백엔드 API 완성 (Core)

> 목표: 프론트가 붙을 수 있는 REST API 완성

### 1-1. DB 초기화 및 데이터 시딩

- [ ] Docker Compose로 PostgreSQL(PostGIS) + Redis 환경 구성
- [ ] `_shared`, `EPL`, `LaLiga`, `Bundesliga`, `SerieA`, `Ligue1` SQL 스크립트 순서대로 실행
- [ ] `combine_sql.py` 검토 및 실행
- [ ] 선수 데이터 크롤러(`fd_client.py`, `tm_client.py`) 실행 → DB INSERT
- [ ] `application.properties` 환경변수 분리 (`.env` 또는 `application-local.properties`)

### 1-2. Service 비즈니스 로직 구현

**JournalistService**
- [ ] X Handle로 기자 조회/등록
- [ ] 공신력 점수 기준으로 기자 랭킹 조회 (Redis 캐시 적용, TTL 30분)
- [ ] 기자별 뉴스 목록 조회

**PostService**
- [ ] X API에서 수집한 트윗 저장
- [ ] 중복 `x_post_id` 체크

**TransferNewsService**
- [ ] POST → TRANSFER_NEWS 파싱/추출 로직
- [ ] 상태(RUMOR/CONFIRMED/DENIED/LOAN) 필터 조회
- [ ] 선수별 이적 히스토리 조회
- [ ] 구단별 이적 인/아웃 조회

**CredibilityMetricService**
- [ ] 공신력 점수 산출
  ```
  credibility_score = speed_score × 0.3 + accuracy_score × 0.5 + impact_score × 0.2
  ```
- [ ] 월별 스냅샷 저장 (`measured_date`)
- [ ] 파급력 점수 계산: `(retweet × 3 + like + view × 0.1) / follower`

### 1-3. REST Controller 작성

```
GET  /api/journalists                    기자 목록 (공신력 순 정렬)
GET  /api/journalists/{id}               기자 상세
GET  /api/journalists/{id}/news          기자별 이적 뉴스

GET  /api/news                           이적 뉴스 피드 (최신순, 페이지네이션)
GET  /api/news?status=CONFIRMED          상태 필터
GET  /api/news?leagueId=1               리그 필터

GET  /api/players/{id}                   선수 상세
GET  /api/players/{id}/transfers         선수 이적 히스토리

GET  /api/clubs/{id}                     구단 상세
GET  /api/clubs/{id}/transfers           구단 이적 인/아웃

GET  /api/leagues                        5대 리그 목록
GET  /api/leagues/{id}/clubs             리그 소속 구단 (지도 마커용 lat/lon 포함)
GET  /api/leagues/{id}/news              리그별 이적 뉴스
```

### 1-4. X API 수집 파이프라인

- [ ] `XApiClient` — 기자 X Handle 기준 타임라인 수집
- [ ] `@Scheduled` 스케줄러: 15분 간격으로 수집 (Redis rate-limit 카운터 연동)
- [ ] 수집된 Post → TransferNews 파싱 (키워드 기반: "signing", "deal", "loan", "fee", "here we go" 등)
- [ ] Redis 캐시 키 적용
  - `news:feed:latest` (TTL 10분)
  - `journalist:ranking` (TTL 30분)
  - `journalist:{id}:score` (TTL 1시간)
  - `x-api:rate-limit` (TTL 15분)

---

## Phase 2 — 프론트엔드 구현

> 목표: 유럽 지도 기반 이적 뉴스 시각화 앱

### 2-1. 프로젝트 세팅

- [ ] React (Vite) + TypeScript 프로젝트 생성 (`D:/Transfer/FE/`)
- [ ] Tailwind CSS 적용
- [ ] Axios + React Query (서버 상태 관리)
- [ ] D3.js (지도 렌더링)

### 2-2. 페이지 구성

**메인 페이지 (`/`)**
- [ ] 유럽 지도 (D3 Mercator, 현재 mock HTML 기준)
- [ ] 5대 리그 로고 마커 (클릭 시 해당 리그 뉴스 패널 열기)
- [ ] 구단 위치 마커 (`CLUB.latitude/longitude`)
- [ ] 이적 경로 시각화 (from_club → to_club 선 연결)
- [ ] TransferMap 타이틀 오버레이

**이적 뉴스 피드 패널**
- [ ] 최신 이적 뉴스 리스트 (우측 슬라이드 패널)
- [ ] 상태 배지 (RUMOR / CONFIRMED / DENIED / LOAN)
- [ ] 기자 이름 + 공신력 배지
- [ ] 리그/선수/구단 필터

**기자 랭킹 페이지 (`/journalists`)**
- [ ] 공신력 점수 순 랭킹 테이블
- [ ] 기자 클릭 → 해당 기자 뉴스 목록
- [ ] 점수 구성 시각화 (speed / accuracy / impact)

**선수 상세 페이지 (`/players/:id`)**
- [ ] 선수 프로필 (사진, 소속, 포지션, 계약 만료일)
- [ ] 이적 히스토리 타임라인

### 2-3. 지도 인터랙션

- [ ] 리그별 국가 강조 (hover 시 fill 변경)
- [ ] 구단 마커 클러스터링 (같은 리그 내 밀집 구단)
- [ ] 이적 경로 애니메이션 (화살표 이동 효과)
- [ ] 줌/패닝

---

## Phase 3 — 인프라 및 배포

### 3-1. Docker 환경 구성

- [ ] `docker-compose.yml` 작성 (PostgreSQL + PostGIS, Redis, Spring Boot, Nginx)
- [ ] `Dockerfile` — BE (multi-stage build)
- [ ] 환경 변수 관리 (`.env.example` 작성)

### 3-2. 운영 배포

- [ ] CI/CD 파이프라인 (GitHub Actions)
  - Push → 빌드 → 테스트 → 이미지 빌드 → 배포
- [ ] 서버 선택 (EC2 / Render / Railway 등)
- [ ] Nginx 리버스 프록시 (FE 정적 파일 서빙 + BE 프록시)
- [ ] HTTPS 설정 (Let's Encrypt)

### 3-3. 모니터링

- [ ] Spring Actuator + 헬스체크 엔드포인트
- [ ] Redis 캐시 히트율 확인
- [ ] X API rate limit 로그

---

## 우선순위 요약

```
Phase 1-1  DB 환경 구성 + 데이터 시딩         ← 지금 당장
Phase 1-2  Service 비즈니스 로직              ← DB 완료 후
Phase 1-3  REST Controller                   ← Service 완료 후
Phase 1-4  X API 수집 파이프라인              ← API 안정화 후
Phase 2    프론트엔드                         ← Phase 1 완료 후
Phase 3    인프라/배포                        ← 기능 완성 후
```

---

## 기술 부채 / 주의사항

| 항목 | 내용 |
|------|------|
| X API 비용 | Basic 티어 한도 내 수집 설계 필요 (rate-limit Redis 카운터 필수) |
| 공신력 점수 가중치 | 초기값 (0.3 / 0.5 / 0.2) — 데이터 쌓인 후 튜닝 필요 |
| 이적 뉴스 파싱 | 자연어 파싱 정확도 한계 → 초기에는 키워드 기반, 이후 LLM 보조 검토 |
| PostGIS | 지도 범위 쿼리 시 공간 인덱스(`GIST`) 필수 |
| TRANSFER_NEWS 자동 추출 | POST → TRANSFER_NEWS 자동화 정확도 낮을 수 있음 → 수동 검수 UI 고려 |
