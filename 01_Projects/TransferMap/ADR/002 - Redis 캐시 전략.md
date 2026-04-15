---
tags: [adr, transfermap]
status: "결정됨"
date: 2026-04-14
---

# ADR-002: Redis 캐시 전략

## 상태
✅ 결정됨

## 컨텍스트
- 기자 공신력 순위 산출은 비용이 큰 집계 쿼리
- 이적 뉴스 피드는 요청 빈도가 높음
- X API rate limit 제어 필요
- Kafka 도입은 현 규모에서 오버엔지니어링

## 결정
**Redis 단일 사용** (캐시 + rate-limit 카운터)

## 캐시 키 설계

| 키 | 내용 | TTL |
|----|------|-----|
| `journalist:ranking` | 공신력 순위 리스트 | 30분 |
| `news:feed:latest` | 최신 이적 뉴스 피드 | 10분 |
| `journalist:{id}:score` | 기자별 공신력 점수 | 1시간 |
| `x-api:rate-limit` | X API 수집 주기 카운터 | 15분 |

## 기각된 대안

| 기술 | 기각 이유 |
|------|-----------|
| Kafka | 단일 수집 파이프라인 → 오버엔지니어링 |
| Spring Cache (in-memory) | 서버 재시작 시 캐시 소멸, 분산 불가 |

## 결과
- 장점: 구성 단순, rate-limit 카운터 겸용
- 주의: TTL 설계가 데이터 신선도에 직접 영향
