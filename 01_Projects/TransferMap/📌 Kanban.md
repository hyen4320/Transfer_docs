---

kanban-plugin: board
tags:
  - transfermap
  - kanban

---

## 🔲 백로그

- [ ] GitHub Actions CI/CD 파이프라인
- [ ] Nginx 리버스 프록시 + HTTPS
- [ ] BE / FE Dockerfile (multi-stage build)
- [ ] 운영 배포 (EC2 / Render / Railway)
- [ ] Cloudflare WAF 규칙 설정
- [ ] Canonical URL (`<link rel="canonical">` 각 페이지 추가)
- [ ] 선수 데이터 크롤러 실행 (fd_client.py, tm_client.py)
- [ ] Filter ★ Save 버튼 — 구현 or 제거 결정
- [ ] UnregisteredMention 큐 구현 결정


## 🔨 진행 중

- [ ] Trending 기능 — BE: TrendingController / TrendingService / @Cacheable
- [ ] Trending 기능 — FE: fetchTrendingPlayers() + SidePanel 연동
- [ ] X API POST → TransferNews 파싱 로직
- [ ] 루머 확정 로직 — DB 저장 스키마 (rumor_status: pending/confirmed/denied)


## 👀 리뷰 / 테스트



## ✅ 완료

- [x] 전체 ERD 설계 (7개 엔티티)
- [x] Domain Model 전체 구현 (Post, Journalist, Club, Player, League, TransferNews, CredibilityMetric, Verification)
- [x] Repository 전체 구현 (Spring Data JPA)
- [x] Service 전체 구현 (JournalistServiceImpl, TransferNewsServiceImpl, CredibilityMetricService 등)
- [x] REST Controller (journalists, news, players, clubs, leagues)
- [x] GlobalExceptionHandler
- [x] Caffeine 캐시 적용 (news:feed TTL 10분, journalist:ranking TTL 30분)
- [x] Flyway 마이그레이션 (V8 — Journalist.isBot 컬럼)
- [x] X API 클라이언트 구조 + 15분 수집 스케줄러 + Redis rate-limit
- [x] CORS 설정 (localhost:5173)
- [x] 5대 리그 SQL 스크립트 작성 + 실행 (EPL, LaLiga, Bundesliga, SerieA, Ligue1)
- [x] 클럽 좌표 수정 (35개 구단 fix_club_coordinates.sql)
- [x] Bundesliga.sql 헤더 오염 텍스트 제거
- [x] Docker Compose (PostgreSQL + PostGIS 16, Redis)
- [x] 유럽 지도 WorldMap (D3 geoMercator, resolveOverlaps)
- [x] WorldMap 모바일 터치 제스처 (pinch zoom + pan + 더블탭 reset)
- [x] WorldMap 성능 최적화 (D3 배칭 54ms→38ms, tooltipPos ref 전환)
- [x] 사이드 패널 (뉴스피드 + 구단 상세 + 이적 탭)
- [x] CountryMapPage (구단 마커 + 모바일 bottom sheet)
- [x] PlayerPanel (이적 히스토리 선 연동, 모바일 bottom sheet, 검색 toast)
- [x] 기자 목록 / 상세 페이지 (공신력 점수, 봇 제외 필터)
- [x] 선수 상세 페이지 + 이적 히스토리
- [x] 에러 페이지 (404 / 500 + ErrorBoundary)
- [x] BE↔FE 데이터 연동 (apiFetch + mapper, FreeAgent 폴백)
- [x] 시즌 연동 (뉴스피드 시즌 선택 → 메인 페이지 마커 동기화)
- [x] react-helmet-async SEO (선수·기자 페이지 동적 OG 태그)
- [x] 파비콘 512×512 PNG + manifest.json 업데이트
- [x] useIsMobile 공용 훅 추출
- [x] AbortController (PlayerPanel, useNewsInfinite)
- [x] React.memo (NewsCard), worldAtlas singleton 캐싱
- [x] LEAGUE_NAME_TO_ID / SEASON_OPTIONS → constants.ts 단일화
- [x] Footer 앵커 분기 (#about, #contact, #privacy)
- [x] Filter Clear 버그 수정 (onClearFilter prop)
- [x] InfoPage 해시 스크롤 useEffect




%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false,false]}
```
%%
