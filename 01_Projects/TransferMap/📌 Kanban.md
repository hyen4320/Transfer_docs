---

kanban-plugin: board
tags:
  - transfermap
  - kanban

---

## 🔲 백로그

- [ ] Docker Compose — PostgreSQL(PostGIS) + Redis 환경 구성
- [ ] 선수 데이터 크롤러 실행 (fd_client.py, tm_client.py)
- [ ] Redis 캐시 키 적용
- [ ] GitHub Actions CI/CD 파이프라인
- [ ] Nginx 리버스 프록시 + HTTPS


## 🔨 진행 중

- [ ] X API 수집 스케줄러 (@Scheduled 15분)
- [ ] React (Vite) + TypeScript 프로젝트 세팅
- [ ] 이적 뉴스 피드 패널 UI
- [ ] 선수 상세 페이지
- [ ] 기자 랭킹 페이지
- [ ] D3.js 유럽 지도 구현
- [ ] Docker Compose 전체 환경 구성


## 👀 리뷰 / 테스트



## ✅ 완료

- [x] 전체 ERD 설계 (7개 엔티티)
- [x] Domain Model 구현
- [ ] JournalistService 구현
- [ ] TransferNewsService 구현
- [ ] CredibilityMetricService 구현
- [ ] REST Controller 작성 (journalists, news, players, clubs, leagues)
- [ ] PostService 구현
- [ ] application.properties 환경변수 분리
- [x] Repository 구현 (6개)
- [x] Service 골격 생성
- [x] X API 클라이언트 구조
- [x] 5대 리그 SQL 스크립트 작성
- [x] 선수 크롤러 구조 (fd/tm)
- [x] 메인 페이지 HTML 디자인 시안
- [ ] SQL 스크립트 실행 (5대 리그 기본 데이터 시딩)




%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false,false]}
```
%%