---
tags: [asset, infra, cicd, github-actions]
status: "🟡 부분 완료"
updated: 2026-04-20
---

# GitHub Actions CI/CD

## 파일 위치

`.github/workflows/ci.yml`

---

## 현재 파이프라인 구성

```
push / PR → main
  ├── be-test    (Java 17, ./gradlew test)
  ├── be-build   (needs: be-test) → docker build transfer-be:{sha}
  └── fe-build   (npm ci && npm run build)
```

### 미완: 배포 스텝 없음

빌드까지만 검증. EC2 배포 스텝이 아직 없음.

---

## 추가해야 할 스텝 (EC2 배포)

```yaml
deploy:
  needs: [be-build, fe-build]
  runs-on: ubuntu-latest
  if: github.ref == 'refs/heads/main'
  steps:
    - name: Push images to registry
      # Docker Hub 또는 ECR에 이미지 push
    - name: SSH into EC2 & deploy
      # appleboy/ssh-action 으로 EC2 접속
      # docker compose pull && docker compose up -d
```

---

## 필요한 GitHub Secrets

| Secret | 설명 |
|--------|------|
| `DB_PASSWORD` | PostgreSQL 비밀번호 |
| `X_API_BEARER_TOKEN` | X API Bearer Token |
| `ANTHROPIC_API_KEY` | Claude API 키 |
| `EC2_HOST` | EC2 퍼블릭 IP 또는 도메인 |
| `EC2_SSH_KEY` | EC2 접속용 PEM 키 (base64) |
| `DOCKER_HUB_USER` | Docker Hub 계정 (또는 ECR) |
| `DOCKER_HUB_TOKEN` | Docker Hub 토큰 |

---

## 상태

- [x] be-test 잡 작성
- [x] be-build 잡 작성
- [x] fe-build 잡 작성
- [ ] Docker 이미지 레지스트리 선택 (Docker Hub vs ECR)
- [ ] deploy 잡 작성 (EC2 SSH 배포)
- [ ] GitHub Secrets 등록
