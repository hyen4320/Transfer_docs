---
tags: [asset, infra, nginx]
status: "✅ 완료"
updated: 2026-04-20
---

# Nginx 리버스 프록시

## 파일 위치

| 파일 | 경로 | 용도 |
|------|------|------|
| FE 내장 nginx.conf | `FE/transfermap/nginx.conf` | FE 컨테이너 내부 정적 파일 서빙 |
| 운영 프록시 설정 | `Infra/nginx/nginx.prod.conf` | `/api` → BE 프록시 |

---

## 라우팅 규칙

```
:80
 ├── /api/*  → http://be:8080  (Spring Boot)
 └── /       → http://fe:80   (React 정적 파일)
```

---

## HTTPS 추가 시 (미완)

도메인 확보 후 Let's Encrypt 적용 예정.

```nginx
server {
    listen 443 ssl;
    ssl_certificate     /etc/letsencrypt/live/{domain}/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/{domain}/privkey.pem;
    ...
}
server {
    listen 80;
    return 301 https://$host$request_uri;
}
```

Certbot 사용:
```bash
certbot --nginx -d {도메인}
```

---

## 상태

- [x] nginx.prod.conf 작성 (/api 프록시 + FE 프록시)
- [x] FE nginx.conf 작성
- [ ] HTTPS 설정 (도메인 확보 후)
- [ ] HTTP → HTTPS 리다이렉트
