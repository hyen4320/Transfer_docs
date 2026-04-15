---
tags: [adr, transfermap]
status: "결정됨"
date: 2026-04-14
---

# ADR-003: X API 수집 전략

## 상태
✅ 결정됨

## 컨텍스트
- X API Basic 티어: 월 50만 트윗 읽기 한도
- 이적 루머는 이적 시장 기간(1~2월, 6~8월)에 집중
- 실시간성보다 주기적 수집으로도 서비스 가능

## 결정
**@Scheduled 15분 간격 폴링 + Redis rate-limit 카운터**

## 수집 흐름
```
@Scheduled(15분)
    → Redis rate-limit 카운터 확인
    → 한도 내: XApiClient.fetchTimeline(xHandle)
    → POST 저장 → 키워드 기반 TRANSFER_NEWS 파싱
    → 한도 초과: 수집 스킵 + 로그
```

## 파싱 키워드 (초기)
`signing`, `deal`, `loan`, `fee`, `here we go`, `medical`, `agreement`, `confirmed`

## 결과
- 장점: 구현 단순, rate-limit 초과 방지
- 주의: 자연어 파싱 정확도 한계 → 데이터 쌓인 후 LLM 보조 검토
