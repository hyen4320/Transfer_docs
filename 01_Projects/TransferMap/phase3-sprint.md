---
tags: [transfermap, planning, sprint]
type: sprint
updated: 2026-05-12
---

# Phase 3 Sprint — 기능 확장 계획

## 우선순위 맵

| # | 기능 | 난이도 | 데이터 의존 | 실행 순서 |
|---|------|--------|------------|----------|
| 20 | Ad Placement Cheatsheet | 🟢 낮음 | 없음 | 1 — 오늘 |
| 30 | Gesture 확장 | 🟢 낮음 | 없음 | 2 — 오늘 |
| 31 | Fixtures Schedule | 🟡 중간 | 외부 API | 3 |
| 32 | Standings | 🟡 중간 | 31과 동일 API | 4 (31과 묶음) |
| 33 | Match Detail | 🔴 높음 | 31/32 안정화 후 | 5 |
| 25–27 | Stadium | 🔴 높음 | 좌표·이미지 수집 | 6 |
| 28–29 | Injury | 🟡 중간 | 데이터 소스 확정 후 | 7 |

---

## 실행 계획

### Step 1 — 데이터 의존 없음 (즉시 시작)

#### 20: 광고 Placement Cheatsheet
- 작업: 페이지별 AdSense 슬롯 위치·사이즈 문서화
- 산출물: [[07_Monetization/광고-placement-cheatsheet]]
- 소요: 반나절

#### 30: Gesture 확장
- 작업: `WorldMap`의 pinch zoom / pan 제스처를 `CountryMapPage`에도 동일하게 적용
- 현재: `WorldMap`에만 `touchstart/touchmove/touchend` 핸들러 구현됨
- 확장 대상: `CountryMapPage` → 동일 `useMapGesture` 훅 추출 후 공유
- 소요: 반나절

---

### Step 2 — 외부 API 연동 (31 + 32 묶음) ✅ BE 완료

> API 선택 근거 → [[ADR/005 - 풋볼 데이터 API 선택]]
> 엔드포인트 레퍼런스 → [[Resources/football-external-api]]
> **현재 API: football-data.org (무료) → 8월 시즌 개막 시 API-Football 유료 전환 예정**

#### 31: Fixtures Schedule (경기 일정) ✅ BE 완료
- 라우트: `/fixtures` (신규 페이지)
- 데이터 흐름: football-data.org → BE 캐시(Redis TTL 5분) → FE
- BE: `FixturesController` + `FixturesServiceImpl` + Redis 캐싱 ← **완료**
- FE: 리그 탭 전환 + 날짜별 경기 카드 목록 + 결과/예정 구분
- 신규 트래픽 유입 효과: 경기 일정 검색 쿼리 SEO 커버

#### 32: Standings (순위표) ✅ BE 완료
- 라우트: `/standings` (신규 페이지) 또는 `/fixtures`와 탭 공유
- 데이터 흐름: football-data.org → BE 캐시(Redis TTL 1시간) → FE
- BE: `FixturesServiceImpl.getStandings()` ← **완료**
- FE: 리그 셀렉터 + 순위 테이블 (팀명·경기수·승·무·패·득실·승점)

---

### Step 3 — 콘텐츠 수집 선행 필요

#### 25–27: Stadium (경기장)
- 데이터 수집: 위키데이터 API + TheSportsDB 이미지
- 수집 대상: 5대 리그 98개 구단 경기장 (이름, 좌표, 수용인원, 이미지 URL)
- 저장: `Stadium` 테이블 신규 (club_id FK)
- FE: 구단 상세 패널에 경기장 카드 추가 + 지도에 경기장 레이어 토글

#### 28–29: Injury (부상 리포트)
- 데이터 소스 결정 필요 → [[ADR/005 - 풋볼 데이터 API 선택]] 참고
- API-Football 사용 시: `/injuries` 엔드포인트 (리그·시즌·팀 파라미터)
- FE: 선수 패널 하단에 부상 상태 배지 + 복귀 예정일

---

### Step 4 — 고난도 (8월 시즌 개막 후, API-Football 결제 후)

#### 33: Match Detail (경기 상세) ⏸ 보류 — API 제약
- **보류 이유:** football-data.org 무료 플랜은 lineups / events 미제공
- **재개 조건:** 8월 API-Football 유료 전환 후 → [[ADR/005 - 풋볼 데이터 API 선택]] 체크리스트 참고
- 라우트: `/fixtures/:fixtureId`
- 데이터: 라인업(11명+교체), 이벤트(골·카드·교체), 통계(점유율 등)
- 렌더링: ISO(격리) 렌더 전략 — 상세 페이지는 별도 lazy chunk로 분리
- BE: `MatchDetailController` → fixtures + lineups + events 통합 응답
- FE: 피치 레이아웃 + 타임라인 이벤트 컴포넌트

---

## 디자인 레퍼런스
→ [[Design/phase3-pages]]

## 변경 이력

| 날짜 | 내용 |
|------|------|
| 2026-05-12 | Phase 3 스프린트 초안 작성 |
| 2026-05-21 | API-Football → football-data.org 임시 전환 (EPL 37R 시즌 종료 임박, 결제 비효율). Step 31/32 BE 완료. Step 33 8월로 보류. |
