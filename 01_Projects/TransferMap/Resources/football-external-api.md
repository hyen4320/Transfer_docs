---
tags: [transfermap, api, reference, external]
type: reference
updated: 2026-05-12
---

# 풋볼 외부 API 레퍼런스

> 선택 근거 → [[ADR/005 - 풋볼 데이터 API 선택]]

## API-Football

- Base URL: `https://v3.football.api-sports.io`
- 인증: `x-apisports-key: {API_KEY}` 헤더
- 무료: 100 req/day, 10 req/min

### Fixtures (경기 일정)

```
GET /fixtures
  ?league={leagueId}
  &season={year}          # e.g. 2024
  &date={YYYY-MM-DD}      # 특정 날짜
  &status=NS              # NS=예정, FT=종료, LIVE=진행중

응답 주요 필드:
  fixture.id, fixture.date, fixture.status.short
  teams.home.name, teams.home.logo
  teams.away.name, teams.away.logo
  goals.home, goals.away
  venue.name, venue.city
```

### Standings (순위표)

```
GET /standings
  ?league={leagueId}
  &season={year}

응답 주요 필드:
  league.standings[0][] →
    rank, team.name, team.logo
    points, goalsDiff
    all.played, all.win, all.draw, all.lose
```

### Injuries (부상)

> 데이터 제공 범위: **2021년 4월 이후**만 제공. 이전 시즌 요청 시 빈 배열 반환.

```
GET /injuries
  ?league={leagueId}
  &season={year}
  &team={teamId}          # 선택

응답 주요 필드:
  player.id, player.name, player.photo
  team.name
  fixture.date            # 부상 보고 시점
  player.type             # "Missing Fixture" | "Questionable"
  player.reason           # 부상 원인
```

### Match Lineups (라인업)

```
GET /fixtures/lineups
  ?fixture={fixtureId}

응답 주요 필드:
  team.name, team.logo, formation
  startXI[].player → id, name, number, pos, grid
  substitutes[].player → 교체 선수
```

### Match Events (이벤트)

```
GET /fixtures/events
  ?fixture={fixtureId}

응답 주요 필드:
  time.elapsed, time.extra
  team.name
  player.name, assist.name
  type            # "Goal" | "Card" | "subst" | "Var"
  detail          # "Normal Goal" | "Yellow Card" | "Red Card" 등
```

### Match Statistics (통계)

```
GET /fixtures/statistics
  ?fixture={fixtureId}

응답 주요 필드 (팀별 배열):
  Ball Possession, Total Shots, Shots on Goal
  Corner Kicks, Fouls, Yellow Cards, Red Cards
  Passes %, Offsides
```

---

## API-Football 리그 ID 매핑

| 리그 | League ID | 시즌 예시 |
|------|-----------|----------|
| EPL | 39 | 2024 |
| La Liga | 140 | 2024 |
| Bundesliga | 78 | 2024 |
| Serie A | 135 | 2024 |
| Ligue 1 | 61 | 2024 |

---

## TheSportsDB (경기장 이미지 수집용)

- Base URL: `https://www.thesportsdb.com/api/v1/json/3`
- 무료 티어: API 키 불필요 (키 "3" 사용)

```
GET /searchteams.php?t={팀명}
  응답: strTeam, strStadium, strStadiumThumb, strStadiumLocation
       intStadiumCapacity, strStadiumDescription

GET /lookupteam.php?id={teamId}
  응답: 동일
```

---

## Wikidata SPARQL (경기장 좌표 수집용)

엔드포인트: `https://query.wikidata.org/sparql`

```sparql
SELECT ?teamLabel ?stadiumLabel ?capacity ?lat ?lon WHERE {
  VALUES ?team { wd:Q18918 wd:Q9617 }   # 구단 Wikidata ID 목록
  ?team wdt:P115 ?stadium .
  ?stadium wdt:P1083 ?capacity .
  ?stadium wdt:P625 ?coord .
  BIND(geof:latitude(?coord) AS ?lat)
  BIND(geof:longitude(?coord) AS ?lon)
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
}
```

---

## BE 연동 구조

```
BE 패키지: com.transfer_map.external
  ├── ApiFootballClient.java      # WebClient 래퍼
  ├── FixturesService.java        # Redis 캐시 레이어
  ├── StandingsService.java
  ├── InjuryService.java
  └── MatchDetailService.java     # lineups + events + statistics 통합
```

환경변수:
```
API_FOOTBALL_KEY=xxx
API_FOOTBALL_BASE_URL=https://v3.football.api-sports.io
```
