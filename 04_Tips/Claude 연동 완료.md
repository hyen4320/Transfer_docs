# Claude 연동 완료

> 작성일: 2026-04-14

---

## Claude MCP란?

**MCP (Model Context Protocol)**는 Claude가 외부 도구 및 서비스와 연동할 수 있도록 하는 프로토콜입니다.  
MCP 서버를 통해 Claude Code에서 Figma, 파일 시스템, 데이터베이스, 웹 검색 등 다양한 기능을 직접 활용할 수 있습니다.

---

## 연동 방법

### 1. Claude Code 설치

```bash
npm install -g @anthropic-ai/claude-code
```

### 2. MCP 서버 설정

Claude Code의 설정 파일(`settings.json`)에 MCP 서버를 등록합니다.

```json
{
  "mcpServers": {
    "서버이름": {
      "command": "npx",
      "args": ["-y", "패키지명"],
      "env": {
        "API_KEY": "your-api-key"
      }
    }
  }
}
```

### 3. 주요 MCP 서버 예시

| MCP 서버 | 기능 |
|----------|------|
| `@anthropic-ai/mcp-server-filesystem` | 로컬 파일 시스템 읽기/쓰기 |
| `plugin:figma:figma` | Figma 디자인 읽기 및 코드 생성 |
| `@modelcontextprotocol/server-brave-search` | 웹 검색 |
| `@modelcontextprotocol/server-github` | GitHub 이슈/PR 관리 |
| `@modelcontextprotocol/server-postgres` | PostgreSQL 쿼리 실행 |

### 4. 연동 확인

Claude Code에서 `/mcp` 명령어로 현재 연결된 MCP 서버 목록을 확인할 수 있습니다.

---

## 활용법

### Figma 연동
- Figma URL을 Claude에 공유하면 디자인을 분석하고 코드를 자동 생성
- `get_design_context` 도구로 컴포넌트 구조 파악
- Code Connect로 Figma 컴포넌트와 실제 코드 컴포넌트를 매핑

### 파일 시스템 연동
- 프로젝트 파일을 직접 읽고 수정 가능
- 코드 분석, 리팩토링, 버그 수정 자동화

### 데이터베이스 연동
- SQL 쿼리 작성 및 실행
- 스키마 분석 및 최적화 제안

### 웹 검색 연동
- 최신 문서 및 정보 검색
- 라이브러리 버전, API 변경사항 실시간 확인

---

## Claude Code 주요 명령어

| 명령어 | 설명 |
|--------|------|
| `/status` | 현재 세션 상태 확인 |
| `/mcp` | MCP 서버 연결 상태 확인 |
| `/help` | 도움말 |
| `/clear` | 대화 컨텍스트 초기화 |
| `/commit` | Git 커밋 생성 |
| `/review-pr` | PR 코드 리뷰 |
| `! <명령어>` | 터미널 명령어 직접 실행 |

---

## Obsidian 사용법

### 기본 개념
- **Vault**: 노트들이 저장되는 최상위 폴더. 모든 `.md` 파일이 노트가 됨
- **노트**: Markdown 형식의 텍스트 파일
- **백링크**: `[[노트이름]]` 형식으로 노트 간 연결

### 핵심 단축키

| 단축키 | 기능 |
|--------|------|
| `Ctrl + N` | 새 노트 생성 |
| `Ctrl + O` | 노트 빠른 열기 |
| `Ctrl + P` | 명령어 팔레트 열기 |
| `Ctrl + E` | 편집/미리보기 모드 전환 |
| `Ctrl + K` | 링크 삽입 |
| `Ctrl + B` | 굵게 |
| `Ctrl + I` | 기울임 |
| `Ctrl + Shift + F` | 전체 검색 |
| `Ctrl + G` | 그래프 뷰 열기 |

### 노트 연결 (링크)

```markdown
[[다른 노트 제목]]           ← 내부 링크
[[노트 제목|표시할 텍스트]]  ← 별칭 링크
![[이미지.png]]              ← 이미지 삽입
![[다른 노트]]               ← 노트 임베드
```

### 태그 활용

```markdown
#태그이름
#상위/하위태그
```

- 우측 패널 → 태그 뷰에서 태그별 노트 모아보기 가능

### 폴더 구조 예시

```
Transfer_vault/
├── 00_Index/        ← 목차, 홈 노트
├── 01_Projects/     ← 진행 중인 프로젝트
├── 02_Areas/        ← 지속 관리 영역 (개발, 학습 등)
├── 03_Resources/    ← 참고 자료
└── 04_Archive/      ← 완료/보관
```

### 유용한 플러그인 (Community Plugins)

| 플러그인           | 용도                       |
| -------------- | ------------------------ |
| **Dataview**   | 노트를 DB처럼 쿼리해서 테이블/리스트 생성 |
| **Templater**  | 노트 템플릿 자동화               |
| **Calendar**   | 날짜별 데일리 노트 관리            |
| **Git**        | vault를 Git으로 버전 관리       |
| **Excalidraw** | 노트 내 다이어그램 작성            |

### Claude와 함께 활용하는 팁
- Claude Code에서 vault 경로를 직접 읽고 노트 생성/수정 가능
- `D:\Transfer\Transfer_vault` 경로로 접근
- 노트 내용 요약, 구조 개선, 태그 추천 등을 Claude에게 요청 가능
- 여러 노트를 한 번에 분석하거나 연결고리 찾기 가능

---

## 참고 링크

- [Claude Code 공식 문서](https://docs.anthropic.com/claude/claude-code)
- [MCP 공식 사이트](https://modelcontextprotocol.io)
- [MCP 서버 목록](https://github.com/modelcontextprotocol/servers)
- [피드백 및 이슈](https://github.com/anthropics/claude-code/issues)
