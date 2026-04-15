---
tags: [adr, transfermap]
status: "결정됨"
date: 2026-04-14
---

# ADR-001: PostgreSQL 선택

## 상태
✅ 결정됨

## 컨텍스트
- 구단 위치 데이터(위도/경도) 기반 지도 범위 쿼리 필요
- 기자 공신력 순위 산출 시 Window Function 필요
- X API 원본 JSON 응답 저장 필요
- 이적 뉴스 본문 검색 기능 필요

## 결정
**PostgreSQL + PostGIS 사용**

## 이유

| 기능 | 활용처 |
|------|--------|
| PostGIS | 구단 위치 마커, 지도 범위 쿼리 |
| Window Function | `RANK() OVER` 기자 순위 산출 |
| JSONB | X API 원본 응답 저장 |
| Full-text Search | 이적 뉴스 본문 검색 |

## 기각된 대안

| 기술 | 기각 이유 |
|------|-----------|
| MySQL | PostGIS 미지원, Window Function 제한적 |
| Elasticsearch | 현 규모에서 PostgreSQL FTS로 충분 |

## 결과
- 장점: PostGIS, Window Function, JSONB 모두 커버
- 주의: 공간 인덱스(`GIST`) 필수 적용 필요
