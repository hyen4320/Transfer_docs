---
tags: [asset, infra, docker]
status: "✅ 완료"
updated: 2026-04-20
---

# Docker 컨테이너 환경

## 파일 위치

| 파일 | 경로 | 용도 |
|------|------|------|
| docker-compose.yml | `Infra/docker-compose.yml` | 로컬 개발 (DB + Redis만) |
| docker-compose.prod.yml | `Infra/docker-compose.prod.yml` | 운영 전체 스택 |
| BE Dockerfile | `BE/Dockerfile` | multi-stage 빌드 |
| FE Dockerfile | `FE/transfermap/Dockerfile` | node build → nginx serve |

---

## 컨테이너 구성 (prod)

| 컨테이너 | 이미지 | 포트 | 비고 |
|---------|--------|------|------|
| transfer-postgres | postgis/postgis:16-3.4-alpine | (내부) | volume: postgres_data |
| transfer-redis | redis:7-alpine | (내부) | volume: redis_data |
| transfer-be | BE/Dockerfile | :8080 (내부) | postgres/redis healthy 대기 |
| transfer-fe | FE/Dockerfile | 80:80, 443:443 | Nginx + FE 정적 파일 |

---

## 주요 환경 변수 (BE)

```
SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/transfer
SPRING_DATASOURCE_USERNAME=${DB_USER}
SPRING_DATASOURCE_PASSWORD=${DB_PASSWORD}
SPRING_DATA_REDIS_HOST=redis
X_API_BEARER_TOKEN=${X_API_BEARER_TOKEN}
ANTHROPIC_API_KEY=${ANTHROPIC_API_KEY}
```

---

## 로컬 전체 스택 기동

```bash
cd D:\Transfer\Infra
cp ../.env.example .env   # 값 채워서 사용
docker compose -f docker-compose.prod.yml up --build
```

---

## 상태

- [x] BE Dockerfile 작성
- [x] FE Dockerfile 작성
- [x] docker-compose.prod.yml 작성
- [ ] 로컬 전체 스택 기동 테스트
- [ ] EC2에서 운영 배포 테스트
