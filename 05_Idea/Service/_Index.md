---
tags: [index, service, idea]
updated: 2026-05-15
---

# Service Ideas — Index

> 서비스 기능 아이디어 및 설계 문서 목록

---

## 구현 완료

| 문서 | 내용 | 비고 |
|---|---|---|
| [[Report 탭]] | 이적시장 분석 리포트 6종 (리그 지출·포지션·구단·이적흐름·TOP5·자유계약) | 기자 신뢰도·이적료 추이는 미구현 |
| [[루머확정로직]] | Admin PATCH로 루머→확정/부인, Player.currentClub 동기화 | 자동 확정(신뢰도 기반)은 미구현 |
| [[리그 순위 테이블]] | 경기 일정·라인업·순위표 (api-football 연동) | 2026-05-15 배포 |

---

## 구현 필요

| 문서 | 내용 | 우선순위 |
|---|---|---|
| [[이적시장 뉴스만 골라내기]] | 규칙 기반 1차 필터 → Claude API 구조화 추출 파이프라인 | 높음 |
| [[조건 검색]] | 기자 필터 추가, 최초 게시자 표현 방식, `CONTRACT_EXTENSION` Status 반영 | 중간 |
| [[부상기록 표시]] | 자료수집 방안·테이블 스키마 설계 필요 | 낮음 |

---

## 분석 / 고민 중

| 문서 | 내용 |
|---|---|
| [[gemini-api-questions]] | Gemini 비용 최적화 (배치·캐싱·모델 선택), tool_use 패턴 질문 정리 |
| [[API 가격 비교]] | (내용 미작성) |
| [[거의 고정값은 DB에서 빼는거 고려할까 말까 고민중]] | (내용 미작성) |
| [[서버 엑세스 로그]] | (내용 미작성) |

---

## 메모

- `조건 검색`: `CONTRACT_EXTENSION` Status enum 반영 아직 안 됨
- `Report 탭`: SSR/pre-rendering 미적용 → 크롤러 접근성 이슈 잠재
- `루머확정로직`: `CredibilityMetric.accuracy_score` 는 Verification 데이터 쌓인 후 계산 가능
