---
tags: [asset, infra, domain, https]
status: "🔲 미결정"
updated: 2026-04-20
---

# 도메인 및 HTTPS

## 현황

도메인 미구매. EC2 생성 후 IP 확인되면 진행.

---

## 도메인 후보

| 후보 | 비고 |
|------|------|
| transfermap.io | 깔끔, 인지도 높은 TLD |
| transfermap.app | HTTPS 강제, 깔끔 |
| transfermap.kr | 한국 서비스 강조 |

> 현재 미결정. 구매 후 이 파일에 확정 도메인 기록.
> transfer-map.com 구매

---

## 구매 추천 레지스트라

| 서비스          | 특징                   |
| ------------ | -------------------- |
| Namecheap    | 가격 저렴, 무료 WhoisGuard |
| AWS Route 53 | EC2와 같은 생태계, 자동화 편리  |
| Cloudflare   | DNS 관리 최강, 무료 CDN 포함 |
gabia에서 산 후 cloudflare로 네임서버를 변경 글로벌 cdn, 무료ssl/tls ddos완화, dns관리, 무제한 대역폭 혜택을 취하긴

---

## DNS 설정 (구매 후)

```
A    @        {EC2_퍼블릭_IP}
A    www      {EC2_퍼블릭_IP}
```

---

## HTTPS 설정 (Let's Encrypt + Certbot)

EC2에서:

```bash
# Certbot 설치
sudo apt install certbot python3-certbot-nginx

# 인증서 발급 (Nginx 플러그인)
sudo certbot --nginx -d transfermap.{tld} -d www.transfermap.{tld}

# 자동 갱신 확인
sudo certbot renew --dry-run
```

인증서 경로:
```
/etc/letsencrypt/live/{domain}/fullchain.pem
/etc/letsencrypt/live/{domain}/privkey.pem
```

`Infra/nginx/nginx.prod.conf`에 HTTPS 블록 추가 필요 → [[nginx]]

---

## Nginx + Docker 시 주의사항

Certbot이 EC2 호스트에 인증서를 발급하면, docker-compose에서 볼륨 마운트로 Nginx 컨테이너에 전달해야 함:

```yaml
fe:
  volumes:
    - /etc/letsencrypt:/etc/letsencrypt:ro
```

---

## 상태

- [x] 도메인 후보 확정
- [x] 도메인 구매
- [ ] DNS A 레코드 연결 (EC2 IP)
- [ ] Let's Encrypt 인증서 발급
- [ ] Nginx HTTPS 설정 적용
- [ ] HTTP → HTTPS 리다이렉트 확인
