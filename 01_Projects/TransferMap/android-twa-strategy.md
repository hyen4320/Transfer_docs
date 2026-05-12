---
tags: [transfermap, android, twa, pwa, deploy]
type: strategy
created: 2026-05-10
status: "🔲 시작 전"
---

# TransferMap Android TWA 배포 전략

> **TWA (Trusted Web Activity):** 배포된 웹사이트를 Chrome 엔진 위에서 전체화면으로 실행하는 안드로이드 앱.
> 코드 수정 = 앱 자동 반영. APK 재배포 불필요.

---

## 전체 흐름

```
Phase 1  서버 배포 (HTTPS 필수)
    ↓
Phase 2  PWA 설정 (manifest.json + Service Worker)
    ↓
Phase 3  TWA 패키징 (Bubblewrap → APK/AAB 빌드)
    ↓
Phase 4  Play Store 등록
```

> TWA는 HTTPS가 없으면 동작하지 않음. Phase 1이 모든 것의 전제.

---

## Phase 1 — 서버 배포

### 현재 상태

| 항목 | 상태 | 비고 |
|------|------|------|
| docker-compose.prod.yml | ✅ 완료 | |
| nginx.prod.conf | ✅ 완료 | |
| BE Dockerfile | ✅ 완료 | multi-stage |
| FE Dockerfile | ✅ 완료 | Nginx 포함 |
| EC2 / 도메인 | ✅ 완료 | transfer-map.com, Cloudflare |
| HTTPS | ✅ 완료 | Cloudflare SSL |

**Phase 1 완료 — Phase 2부터 시작.**

### 1-1. BE Dockerfile 작성

`BE/Dockerfile` (multi-stage)

```dockerfile
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app
COPY gradlew .
COPY gradle gradle
COPY build.gradle settings.gradle ./
COPY src src
RUN chmod +x gradlew && ./gradlew bootJar -x test

FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### 1-2. FE Dockerfile 작성

`FE/transfermap/Dockerfile` (Nginx로 정적 파일 서빙)

```dockerfile
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build:client

FROM nginx:alpine
COPY --from=build /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

`FE/transfermap/nginx.conf` (컨테이너 내부용, SPA 라우팅)

```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

> SSR은 일단 보류. TWA 배포 후 SEO 필요 시 추가.

### 1-3. nginx.prod.conf HTTPS 추가

`Infra/nginx/nginx.prod.conf`에 SSL 블록 추가 필요.

```nginx
upstream be {
    server be:8080;
}

server {
    listen 80;
    server_name transfermap.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name transfermap.example.com;

    ssl_certificate     /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    # TWA 필수: assetlinks.json
    location = /.well-known/assetlinks.json {
        root /usr/share/nginx/html;
        add_header Content-Type application/json;
    }

    location /api/ {
        proxy_pass http://be;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location = /sitemap-players.xml {
        proxy_pass http://be/api/sitemap-players;
        proxy_set_header Host $host;
    }

    location / {
        proxy_pass http://fe:80;
        proxy_set_header Host $host;
    }
}
```

### 1-4. EC2 세팅 순서

```bash
# 1. EC2 t3.small (Ubuntu 22.04) 생성
# 2. 보안 그룹: 80, 443, 22 오픈
# 3. 도메인 A 레코드 → EC2 퍼블릭 IP

# 4. 서버에서
sudo apt update && sudo apt install -y docker.io docker-compose certbot
sudo systemctl enable docker

# 5. Let's Encrypt (컨테이너 띄우기 전에 먼저 발급)
sudo certbot certonly --standalone -d transfermap.example.com

# 6. 인증서 위치 확인: /etc/letsencrypt/live/transfermap.example.com/
# docker-compose.prod.yml에 이미 /etc/nginx/ssl 마운트 설정 있음
# 심볼릭 링크 또는 복사 필요:
sudo ln -s /etc/letsencrypt/live/transfermap.example.com/fullchain.pem /etc/nginx/ssl/fullchain.pem
sudo ln -s /etc/letsencrypt/live/transfermap.example.com/privkey.pem /etc/nginx/ssl/privkey.pem

# 7. 배포
cd /opt/transfer
git pull
docker-compose -f Infra/docker-compose.prod.yml up -d --build
```

### 1-5. 확인

- `https://도메인/api/news` → 200
- `https://도메인/` → FE 로딩
- Chrome DevTools → 주소창 자물쇠 확인

---

## Phase 2 — PWA 설정

> TWA는 PWA 기준을 충족해야 함. 핵심: manifest.json + Service Worker.

### 2-1. manifest.json 생성

`FE/transfermap/public/manifest.json`

```json
{
  "name": "TransferMap",
  "short_name": "TransferMap",
  "description": "유럽 축구 이적시장 실시간 지도",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#0f172a",
  "theme_color": "#0f172a",
  "orientation": "portrait",
  "lang": "ko",
  "icons": [
    {
      "src": "/favicon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "any maskable"
    },
    {
      "src": "/favicon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "any maskable"
    }
  ]
}
```

> favicon-512.png 필요. 현재 192/48만 있음 → 512px 아이콘 추가 필요.

`FE/transfermap/index.html` `<head>`에 추가:

```html
<link rel="manifest" href="/manifest.json" />
<meta name="theme-color" content="#0f172a" />
<meta name="mobile-web-app-capable" content="yes" />
```

### 2-2. Service Worker 기본 설정

TWA 설치 조건을 충족하는 최소 SW. `FE/transfermap/public/sw.js`

```js
self.addEventListener('install', () => self.skipWaiting());
self.addEventListener('activate', e => e.waitUntil(clients.claim()));

self.addEventListener('fetch', (e) => {
  // 기본 네트워크 우선 전략 — 캐싱 전략은 추후 추가
  e.respondWith(fetch(e.request).catch(() => caches.match(e.request)));
});
```

`FE/transfermap/src/main.tsx`에 SW 등록:

```ts
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/sw.js');
  });
}
```

### 2-3. Lighthouse 체크

Chrome DevTools → Lighthouse → PWA 항목 전부 초록색 확인.
핵심 통과 항목:
- manifest.json 유효
- Service Worker 등록
- HTTPS
- 192px 이상 아이콘

---

## Phase 3 — TWA 패키징

> Google의 공식 CLI `bubblewrap` 사용. Android Studio 불필요.

### 3-1. 환경 준비

```bash
# Node.js 18+ 필요
npm install -g @bubblewrap/cli

# Java 17 JDK 설치 확인
java -version

# Android SDK 설치 (bubblewrap init 시 자동 다운로드 옵션 있음)
```

### 3-2. Bubblewrap 초기화

```bash
mkdir transfermap-twa && cd transfermap-twa
bubblewrap init --manifest https://transfermap.example.com/manifest.json
```

설정 대화형 입력값:

| 항목 | 값 |
|------|-----|
| Application ID | `com.transfermap.app` |
| Host | `transfermap.example.com` |
| Start URL | `/` |
| Display | `standalone` |
| Status bar color | `#0f172a` |
| Splash screen color | `#0f172a` |
| Icon URL | `https://transfermap.example.com/favicon-512.png` |

### 3-3. assetlinks.json 등록

TWA의 핵심 — 서버와 앱의 신뢰 연결.

```bash
# 키스토어 생성 (bubblewrap build 시 자동 생성됨)
bubblewrap build

# SHA-256 fingerprint 확인
keytool -list -v -keystore <키스토어 경로> -alias android
# 또는
bubblewrap fingerprint add
```

`FE/transfermap/public/.well-known/assetlinks.json`

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.transfermap.app",
    "sha256_cert_fingerprints": [
      "AA:BB:CC:DD:..." 
    ]
  }
}]
```

> `sha256_cert_fingerprints`는 `bubblewrap build` 후 실제 fingerprint로 교체.
> 이 파일이 `https://transfermap.example.com/.well-known/assetlinks.json`으로 접근 가능해야 함.

### 3-4. APK/AAB 빌드 및 테스트

```bash
# AAB 빌드 (Play Store용)
bubblewrap build

# 실기기 테스트 (USB 연결)
bubblewrap install
```

실기기에서 주소창이 없으면 TWA 연결 성공.
주소창이 보이면 assetlinks.json 문제.

---

## Phase 4 — Play Store 등록

### 4-1. 준비물

- Google Play Console 개발자 계정 ($25 일회성)
- AAB 파일 (`app-release.aab`)
- 앱 아이콘: 512×512 PNG (배경 없음)
- 스크린샷: 핸드폰 2장 이상 (1080×1920 권장)
- 스토어 설명 (한국어)

### 4-2. Play Console 등록 순서

```
1. play.google.com/console → 새 앱 만들기
2. 앱 이름: TransferMap
3. 앱 번들(AAB) 업로드 → 내부 테스트 트랙
4. 스토어 등록 정보 입력 (설명, 스크린샷, 아이콘)
5. 콘텐츠 등급 설문
6. 가격 및 배포: 무료 / 대한민국
7. 내부 테스트 → 실기기 확인 → 프로덕션 출시
```

> 프로덕션 심사는 영업일 기준 2~7일 소요.

---

## 작업 체크리스트

### Phase 1 — 서버 배포
- [x] `BE/Dockerfile` 작성
- [x] `FE/transfermap/Dockerfile` 작성
- [x] `Infra/nginx/nginx.prod.conf` 설정
- [x] EC2 + 도메인 (transfer-map.com, Cloudflare)
- [x] HTTPS 적용

### Phase 2 — PWA
- [ ] `favicon-512.png` 추가 (512×512px)
- [ ] `manifest.json` 생성
- [ ] `index.html`에 manifest 링크 추가
- [ ] `sw.js` Service Worker 작성
- [ ] `main.tsx`에 SW 등록 코드 추가
- [ ] Lighthouse PWA 전 항목 통과

### Phase 3 — TWA
- [ ] Bubblewrap CLI 설치
- [ ] `bubblewrap init` 실행
- [ ] `bubblewrap build` → 키스토어 생성
- [ ] SHA-256 fingerprint 확인
- [ ] `assetlinks.json` 작성 및 서버 배포
- [ ] `/.well-known/assetlinks.json` 접근 확인
- [ ] 실기기 TWA 설치 테스트 (주소창 없는지 확인)

### Phase 4 — Play Store
- [ ] Play Console 개발자 계정 생성 ($25)
- [ ] 스크린샷 촬영 (최소 2장)
- [ ] 스토어 등록 정보 작성
- [ ] AAB 업로드 → 내부 테스트
- [ ] 프로덕션 출시 신청

---

## 주의사항

| 항목 | 내용 |
|------|------|
| assetlinks.json | SHA-256이 틀리면 TWA 연결 실패, 주소창 노출됨 |
| 인증서 갱신 | Let's Encrypt 90일 만료 → cron으로 `certbot renew` 자동화 필요 |
| favicon-512.png | maskable 아이콘 권장 (safe zone 80% 내에 로고 배치) |
| Play Store 심사 | 첫 심사 2~7일 소요, 업데이트는 수 시간 내 반영 |
| SSR 비활성화 | Dockerfile에서 `build:client`만 사용 (SSR은 별도 Node 서버 필요, TWA 단계에서는 보류) |
