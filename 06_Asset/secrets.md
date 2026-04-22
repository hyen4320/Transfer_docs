---
tags: [asset, infra, secrets, env]
status: "✅ 완료"
updated: 2026-04-20
---

# 환경 변수 / 시크릿 관리

## 파일 위치

| 파일 | 경로 | 비고 |
|------|------|------|
| 예시 파일 | `.env.example` | 커밋됨, 실제 값 없음 |
| 실제 값 | `.env` (gitignore) | 로컬/서버에서 직접 생성 |
| BE 설정 예시 | `BE/src/main/resources/application.properties.example` | 커밋됨 |
| BE 실제 설정 | `BE/src/main/resources/application.properties.secrets` | gitignore |

---

## 환경 변수 목록

| 변수 | 용도 | 출처 |
|------|------|------|
| `DB_USER` | PostgreSQL 유저 (기본: postgres) | 직접 설정 |
| `DB_PASSWORD` | PostgreSQL 비밀번호 | 직접 설정 |
| `X_API_BEARER_TOKEN` | X(Twitter) API 수집용 | X Developer Portal |
| `ANTHROPIC_API_KEY` | Claude API (트윗 파싱) | Anthropic Console |

---

## 키 발급 위치

| 키 | 발급 위치 |
|----|----------|
| X API Bearer Token | https://developer.x.com → 프로젝트 → Keys and Tokens |
| Anthropic API Key | https://console.anthropic.com → API Keys |

---

## 로컬 설정 방법

```bash
# 프로젝트 루트
cp .env.example .env
# .env 열어서 실제 값 입력

# BE
cp BE/src/main/resources/application.properties.example \
   BE/src/main/resources/application.properties
# application.properties 열어서 anthropic.api-key 등 입력
```

---

## 서버(EC2) 설정 방법

```bash
# EC2 접속 후
nano ~/transfer/.env   # 값 입력
docker compose -f docker-compose.prod.yml up -d
```

---

## 남은 것

- [ ] `anthropic.api-key` application.properties에 입력
- [ ] EC2 서버에 `.env` 배포
- [ ] GitHub Secrets에 CI/CD용 변수 등록 → [[cicd]]
