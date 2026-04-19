---
tags: [adr, transfermap]
status: "결정됨"
date: 2026-04-14
updated: 2026-04-18
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
@Scheduled(fixedDelay=15분)
    → Redis rate-limit 카운터 확인 (15분 윈도우 당 50 요청 한도)
    → 한도 내: XApiClient.getUserTweets(xHandle, sinceId)
        - sinceId: DB에 저장된 가장 최근 포스트의 xPostId (없으면 null → 전체)
        - 응답 트윗을 Post 엔티티로 변환 후 중복 체크(existsByXPostId)하여 저장
    → 한도 초과: 수집 중단 + warn 로그
```

## 패키지 위치
- `transfer.be.api.XApiClient` — X API HTTP 클라이언트
- `transfer.be.api.XApiConfig` / `XApiProperties` — Bearer Token 설정
- `transfer.be.scheduler.XCollectorScheduler` — 스케줄러 (`@ConditionalOnBean`으로 현재 비활성화, 완전 구현 시 `@Scheduled` 적용 예정)
- `transfer.be.service.impl.PostServiceImpl` — since_id 증분 수집 및 저장 로직

## TransferNews 파싱 (미구현)
수집(Post 저장)과 파싱(TransferNews 생성)은 **분리된 단계**로 설계됨.
현재는 Post만 수집하며, TransferNews 레코드는 수동 등록 또는 별도 파싱 단계에서 생성.

파싱 키워드 후보 (추후 구현 시):
`signing`, `deal`, `loan`, `fee`, `here we go`, `medical`, `agreement`, `confirmed`

TransferNews.Status 단계: `INTEREST → RUMOR → CONFIRMED / DENIED / LOAN`

## 결과
- 장점: 구현 단순, since_id로 중복 수집 방지, rate-limit 초과 방지
- 현재 스케줄러 비활성화 상태: 완전 구현 완료 후 `@Scheduled` 활성화 예정
- 주의: 자연어 파싱 정확도 한계 → 데이터 쌓인 후 LLM 보조 검토
