# Obsidian Plugin 활용법

> 작성일: 2026-04-14

---

## 플러그인 설치 방법

`설정(⚙️)` → `Community plugins` → `Browse` → 검색 후 Install → Enable

---

## 생산성 핵심 플러그인

### Dataview
노트를 데이터베이스처럼 쿼리해서 동적 목록/테이블 생성.

```dataview
TABLE file.ctime AS "생성일", tags AS "태그"
FROM #프로젝트
SORT file.ctime DESC
```

- 태그, 폴더, 날짜 기준으로 노트 자동 집계
- 프로젝트 현황판, 독서 목록, 할일 대시보드 등에 활용

---

### Templater
날짜, 파일명, 커서 위치 등을 변수로 쓰는 고급 템플릿.

```markdown
---
title: <% tp.file.title %>
date: <% tp.date.now("YYYY-MM-DD") %>
tags: []
---

# <% tp.file.title %>

## 요약

## 내용

```

- `Ctrl + Alt + N`으로 템플릿 기반 노트 즉시 생성
- 폴더별로 자동 적용 템플릿 지정 가능

---

### Calendar
데일리 노트를 달력 뷰로 관리.

- 우측 패널에 월 달력 표시
- 날짜 클릭 → 해당 날의 데일리 노트 바로 생성/열기
- Periodic Notes 플러그인과 함께 써서 주간/월간 노트도 관리

---

### Git
vault 전체를 Git으로 버전 관리 및 자동 백업.

- 설정에서 자동 커밋 주기 지정 (예: 10분마다)
- GitHub private repo에 push해서 기기 간 동기화
- 변경 이력으로 노트 복구 가능

**설정 예시:**
```
Auto commit interval: 10 (분)
Auto push: ON
Commit message: "vault backup: {{date}}"
```

---

### Excalidraw
노트 안에서 바로 손그림 스타일 다이어그램 작성.

- `[[다이어그램.excalidraw]]` 형식으로 노트에 임베드
- 아키텍처 스케치, 플로우차트, 마인드맵 등에 활용
- 양방향 링크 지원 — 다이어그램 안 텍스트를 노트로 연결 가능

---

### Kanban
노트 기반 칸반 보드.

```markdown
## 백로그
- [ ] 기능 A 설계

## 진행 중
- [ ] 기능 B 구현

## 완료
- [x] 기능 C 테스트
```

- 태스크를 드래그 앤 드롭으로 컬럼 이동
- 각 카드를 독립 노트로 연결 가능

---

### Obsidian Copilot
ChatGPT/Claude 등 LLM을 vault 안에서 직접 사용.

- 노트 내용 기반으로 질문/요약/번역
- vault 전체를 컨텍스트로 삼아 답변 생성
- OpenAI, Anthropic API 키 연동

---

### Various Completes (자동완성)
`[[` 입력 시 노트 제목 자동완성, 태그 자동완성 등 제공.

---

## 플러그인 조합 추천

| 목적 | 추천 조합 |
|------|-----------|
| 프로젝트 관리 | Kanban + Dataview + Templater |
| 개인 지식베이스 | Dataview + Templater + Calendar |
| 백업/동기화 | Git + Remotely Save |
| 개발 노트 | Excalidraw + Dataview + Code Block |
| AI 활용 | Copilot + Templater |

---

## 주의사항

- Community 플러그인은 서드파티 코드 — 민감한 정보가 있는 vault라면 신뢰할 수 있는 플러그인만 설치
- 플러그인이 많을수록 앱 시작 속도 저하 가능 → 안 쓰는 건 비활성화
- Obsidian 업데이트 후 플러그인 호환성 오류 날 수 있으니 자동 업데이트 주의
