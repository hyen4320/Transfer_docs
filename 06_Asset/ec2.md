---
tags: [asset, infra, aws, ec2]
status: "🔲 미생성"
updated: 2026-04-20
---

# AWS EC2 서버

## 현황

생성
docker-compose.prod.yml 로컬 테스트 해봐야함

---

## 권장 스펙

| 항목      | 권장값                   | 비고                       |
| ------- | --------------------- | ------------------------ |
| 인스턴스 타입 | t3.small (2vCPU, 2GB) | 최소. DB+Redis+BE+FE 4컨테이너 |
| OS      | Ubuntu 24.04 LTS      | Docker 공식 지원             |
| 스토리지    | 20GB gp3              | PostgreSQL 볼륨 포함         |
| 리전      | ap-northeast-2 (서울)   | 국내 지연 최소화                |

> t3.micro(1GB)는 PostgreSQL+Spring Boot 동시 기동 시 OOM 위험.

---

## 생성 절차

1. AWS 콘솔 → EC2 → Launch Instance
2. AMI: Ubuntu 24.04 LTS
3. 인스턴스 타입: t3.small
4. 키 페어 생성 → PEM 파일 저장 (분실 불가)
5. 보안 그룹:
   - 인바운드 22 (SSH) — 내 IP만
   - 인바운드 80 (HTTP) — 0.0.0.0/0
   - 인바운드 443 (HTTPS) — 0.0.0.0/0
6. 탄력적 IP 할당 → 인스턴스에 연결

---

## 초기 서버 세팅

```bash
# EC2 접속
ssh -i transfer-key.pem ubuntu@{EC2_IP}

# Docker 설치
curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker ubuntu

# 프로젝트 clone
git clone https://github.com/{user}/Transfer.git ~/transfer
cd ~/transfer

# 환경 변수 설정
cp .env.example .env
nano .env   # 실제 값 입력

# 전체 스택 기동
docker compose -f Infra/docker-compose.prod.yml up -d
```

---

## 비용 (예상, 서울 리전 기준)

| 항목 | 월 비용 |
|------|---------|
| t3.small On-Demand | ~$17 |
| 탄력적 IP (연결 중) | 무료 |
| gp3 스토리지 20GB | ~$1.6 |
| 트래픽 (소규모) | ~$1 이하 |
| **합계** | **~$20/월** |

> 절감: Reserved Instance 1년 약정 시 ~40% 할인 (~$12/월)

---

## 상태

- [ ] EC2 인스턴스 생성
- [ ] 보안 그룹 설정
- [ ] 탄력적 IP 할당
- [ ] Docker 설치 및 초기 세팅
- [ ] docker-compose.prod.yml 운영 배포 테스트
- [ ] CI/CD 배포 스텝 연결 → [[cicd]]
