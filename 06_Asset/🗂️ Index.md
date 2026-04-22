---
tags: [asset, infra, transfermap]
updated: 2026-04-20
---

# 인프라 자산 목록

TransferMap 운영에 필요한 모든 인프라 자산을 관리하는 디렉토리.

---

## 자산 현황

| 자산 | 상태 | 파일 |
|------|------|------|
| Docker 컨테이너 환경 | ✅ 작성 완료 | [[docker]] |
| GitHub Actions CI/CD | ✅ 작성 완료 | [[cicd]] |
| Nginx 리버스 프록시 | ✅ 작성 완료 | [[nginx]] |
| 환경 변수 / 시크릿 | ✅ 정리 완료 | [[secrets]] |
| AWS EC2 서버 | 🔲 미생성 | [[ec2]] |
| 도메인 | 🔲 미결정 | [[domain]] |
| HTTPS / TLS | 🔲 미설정 | [[domain]] |

---

## 배포 구조

```
인터넷
  ↓ :80 / :443
[EC2] ── Nginx (FE static + /api 프록시)
           ├── transfer-fe  (nginx:alpine, :80)
           ├── transfer-be  (temurin:17-jre, :8080)
           ├── transfer-postgres (postgis:16-3.4-alpine)
           └── transfer-redis    (redis:7-alpine)
```

---

## 남은 작업

- [ ] EC2 인스턴스 생성 → [[ec2]]
- [ ] 도메인 구매 및 DNS 연결 → [[domain]]
- [ ] HTTPS 인증서 발급 (Let's Encrypt) → [[domain]]
- [ ] CI/CD에 EC2 배포 스텝 추가 → [[cicd]]
- [ ] docker-compose.prod.yml 로컬 전체 스택 기동 테스트
