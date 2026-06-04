---
tags: [transfermap, adsense, monetization]
type: cheatsheet
updated: 2026-05-12
---

# 광고 Placement Cheatsheet

> AdSense 슬롯 배치 기준 문서. 구현 시 이 문서를 기준으로 컴포넌트 위치 결정.

---

## 기본 원칙

- 콘텐츠 위에 광고가 올라오지 않도록 (above-the-fold 점령 금지)
- 클릭 유도 레이아웃 금지 (AdSense 정책)
- 모바일: 세로 스크롤 흐름에 자연스럽게 삽입
- 지도 위에 광고 오버레이 금지 (UX 파괴)

---

## 페이지별 슬롯 배치

### `/` 메인 (지도)

| 위치 | 슬롯 타입 | 사이즈 | 비고 |
|------|----------|--------|------|
| 뉴스 피드 패널 — 4번째 카드 아래 | In-feed | 300×250 | 스크롤 중 자연 노출 |
| 뉴스 피드 패널 — 10번째 카드 아래 | In-feed | 300×250 | 2번째 슬롯 |

- 지도 영역: 광고 없음
- 패널 상단 고정 배너: 없음 (UX 방해)

---

### `/fixtures` 경기 일정

| 위치 | 슬롯 타입 | 사이즈 | 비고 |
|------|----------|--------|------|
| 날짜 네비게이터 아래 (리스트 최상단) | Leaderboard | 728×90 (데스크톱) / 320×50 (모바일) | 페이지 진입 시 바로 노출 |
| 경기 카드 5번째 아래 | In-feed | 300×250 | 스크롤 흐름 내 삽입 |

---

### `/standings` 순위표

| 위치 | 슬롯 타입 | 사이즈 | 비고 |
|------|----------|--------|------|
| 리그 탭 아래, 테이블 위 | Leaderboard | 728×90 (데스크톱) / 320×50 (모바일) | |
| 테이블 아래 (하단) | Rectangle | 300×250 | 테이블 다 본 후 |

---

### `/players/:id` 선수 상세

| 위치 | 슬롯 타입 | 사이즈 | 비고 |
|------|----------|--------|------|
| 프로필 헤더와 이적 히스토리 사이 | Rectangle | 300×250 | 자연 구분선 역할 |
| 관련 뉴스 목록 3번째 카드 아래 | In-feed | 300×250 | |

---

### `/journalists/:id` 기자 상세

| 위치 | 슬롯 타입 | 사이즈 | 비고 |
|------|----------|--------|------|
| 프로필 헤더 아래 | Rectangle | 300×250 | |

---

### `/fixtures/:fixtureId` 경기 상세

| 위치 | 슬롯 타입 | 사이즈 | 비고 |
|------|----------|--------|------|
| 탭 컨텐츠 하단 (라인업/통계 아래) | Rectangle | 300×250 | 콘텐츠 소비 후 |

---

## 슬롯 ID 관리

```ts
// src/constants/adsense.ts
export const ADSENSE_SLOTS = {
  NEWS_FEED_1: 'XXXXXXXX',   // 뉴스피드 4번째
  NEWS_FEED_2: 'XXXXXXXX',   // 뉴스피드 10번째
  FIXTURES_TOP: 'XXXXXXXX',
  FIXTURES_INLINE: 'XXXXXXXX',
  STANDINGS_TOP: 'XXXXXXXX',
  STANDINGS_BOTTOM: 'XXXXXXXX',
  PLAYER_DETAIL_MID: 'XXXXXXXX',
  PLAYER_DETAIL_FEED: 'XXXXXXXX',
  JOURNALIST_DETAIL: 'XXXXXXXX',
  MATCH_DETAIL: 'XXXXXXXX',
} as const
```

---

## 광고 컴포넌트

```tsx
// src/components/AdSlot.tsx
// data-ad-slot prop으로 슬롯 ID 주입
// 모바일/데스크톱 사이즈 자동 분기
```

---

## 체크리스트

- [ ] AdSense 계정에서 슬롯 ID 발급
- [ ] `ADSENSE_SLOTS` 상수에 실제 ID 채우기
- [ ] `AdSlot` 컴포넌트 구현
- [ ] 페이지별 삽입 위치에 `<AdSlot slot={...} />` 추가
- [ ] 모바일 레이아웃 깨짐 없는지 확인
- [ ] AdSense 정책 위반 여부 최종 검토
