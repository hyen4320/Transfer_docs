---
tags: [idea, service, adsense, content]
created: 2026-05-03
updated: 2026-05-03
status: 구현완료 (1차)
---

# Report 탭 — 이적시장 분석 보고서

AdSense 승인을 위한 고유 텍스트 콘텐츠 확보 + 서비스 자체 가치 향상이 목적.

---

## 구현 현황 (2026-05-03)

### 완료된 것

**BE**
- `GET /api/report/league-spending?season=` → 리그별 이적 지출
- `GET /api/report/position-trend?season=` → 포지션별 뉴스 수
- `GET /api/report/club-activity?season=` → 구단별 영입 활성도 (상위 20)
- `GET /api/report/transfer-flow?season=` → 국가 간 이적 흐름 (상위 20, 국내 이적 제외)
- `GET /api/report/top-deals?season=` → 최고액 확정 이적 TOP 5
- `GET /api/report/free-agent?season=` → 자유계약 이적 리그별 집계
- `TransferNewsRepository` — 집계 JPQL 쿼리 6개 추가
- `ReportService` / `ReportServiceImpl` / `ReportController` / DTO 6개 생성

**FE**
- `src/api/report.ts` — API 레이어
- `src/components/ReportPage.tsx` — 카드뉴스 형태 6카드 레이아웃 (각 카드: accent 색상, hero stat, 설명 문단, 미니 데이터 목록)
- `/report` 라우트 추가 (App.tsx)
- 데스크톱 Topbar에 **Report** 메뉴 추가
- MobileTabBar MORE 메뉴에 **Report** 추가
- `sitemap.xml`에 `/report` 추가 (priority 0.9)

---

## 진행 방안

### 구조

- 메인 네비게이션에 **Report** 탭 추가 ✅
- `/report` — 단일 URL, 시즌 선택으로 데이터 전환 ✅
- 리포트 페이지는 **텍스트 중심** 구성 + 표 보조 ✅

### 리포트 종류

| 리포트 | 내용 | 구현 |
|---|---|---|
| 리그별 이적 지출 현황 | CONFIRMED 딜 기준 지출 합계 | ✅ |
| 포지션별 영입 트렌드 | GK/DF/MF/FW 뉴스 비중 | ✅ |
| 구단별 거래 활성도 | 영입 보고 건수 기준 상위 20개 | ✅ |
| 국가 간 이적 흐름 | 국경 간 이적 상위 20개 경로 | ✅ |
| 기자 신뢰도 리포트 | 신뢰도 상위 기자, 루머→확정 전환율 | 미구현 |
| 이적료 추이 | 시즌별 평균/최고 이적료 | 미구현 |
| 최고액 확정 이적 TOP5 | 시즌 최대 이적료 딜 순위 | ✅ |
| 자유계약 현황 | fromClub=null 이적, 리그별 집계 | ✅ |
### 콘텐츠 작성 방식

- **자동 생성 수치 + 정적 분석 문단** 조합 ✅
- 각 섹션마다 설명 문단 삽입 (구글 크롤러용 텍스트)

---

## 발전 방향

### 단기

- 시즌 종료 후 **시즌 총정리 리포트** 자동 생성 (수치) + 수동 분석 추가
- 리포트 페이지에 **sitemap.xml** 반영 → Search Console 색인 요청

### 중기

- **구단 상세 페이지** (`/club/manchester-city`) — 구단별 이적 히스토리 + 지출 통계
- **선수 프로필 페이지** (`/player/haaland`) — 이적 타임라인 + 관련 루머 이력
- 위 페이지들이 생기면 크롤러가 읽을 수 있는 URL이 수십~수백 개로 늘어남

### 장기

- 리포트를 RSS 피드로 제공 → 외부 유입 채널 확보
- 리포트 공유 기능 (SNS 마케팅 콘텐츠로 활용)

---

## 주의사항

### 크롤러 접근성 (가장 중요)

- 현재 React SPA 구조는 JS 없이는 콘텐츠가 비어있음
- 구글봇은 JS를 렌더링하지만 **느리고 불완전**함
- 리포트 페이지는 **SSR(서버사이드 렌더링) 또는 pre-rendering** 적용을 검토
  - Vite 기반이면 `vite-plugin-ssr` 또는 Next.js 마이그레이션 고려
  - 단기 대안: `react-snap`으로 정적 HTML 미리 생성

### 콘텐츠 품질

- 숫자만 나열하는 리포트는 효과 없음 — 반드시 분석 문장 포함
- 리포트마다 **고유한 제목 + meta description** 필수
- 업데이트 날짜 명시 (구글은 최신성을 중요하게 봄)

### AdSense 정책

- 리포트 페이지에도 광고 슬롯 배치 가능하나, 콘텐츠 대비 광고 비율 주의
- 텍스트가 적은 리포트에 광고가 많으면 또 다른 거절 사유가 됨

---

## 연관 문서

- [[애드센스 거절 원인 분석]]
- [[광고 최적화]]
