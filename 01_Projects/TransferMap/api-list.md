---
tags: [transfer, backend, api, rest]
created: 2026-04-15
updated: 2026-04-19
---

# API 목록

Base URL: `http://localhost:8080`

---

## 뉴스 (News)

### `GET /api/news`
이적 뉴스 복합 조건 검색 (최신순, 페이지네이션)

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `status` | `String` | N | 상태 필터 (`INTEREST` / `RUMOR` / `CONFIRMED` / `DENIED` / `LOAN` / `CONTRACT_EXTENSION`) |
| `season` | `Short` | N | 시즌 인코딩 (`49` = 24/25, `51` = 25/26) |
| `window` | `String` | N | 이적 윈도우 (`SUMMER` / `WINTER`) |
| `leagueId` | `Long` | N | 리그 ID 필터 (toClub 기준) |
| `journalistId` | `Long` | N | 기자 ID 필터 |
| `position` | `String` | N | 선수 포지션 (`GK` / `DF` / `MF` / `FW`) |
| `nationality` | `String` | N | 선수 국적 |
| `minFeeEur` | `Long` | N | 이적료 하한 (유로) |
| `maxFeeEur` | `Long` | N | 이적료 상한 (유로) |
| `from` | `LocalDate` | N | 보도일 시작 (`yyyy-MM-dd`) |
| `to` | `LocalDate` | N | 보도일 종료 (`yyyy-MM-dd`) |
| `toClubId` | `Long` | N | 영입 구단 ID |
| `fromClubId` | `Long` | N | 방출 구단 ID |
| `minReliability` | `Byte` | N | 뉴스 신뢰도 하한 (1~5) |
| `minCredibility` | `Float` | N | 기자 공신력 점수 하한 |
| `verified` | `Boolean` | N | `true` 시 검증된 뉴스만 |
| `page` | `int` | N | 페이지 번호 (기본값 0) |
| `size` | `int` | N | 페이지 크기 (기본값 20) |
| `sort` | `String` | N | 정렬 기준 (예: `feeEur,desc`) |

**Response** `Page<TransferNewsResponse>`

```json
{
  "content": [
    {
      "id": 1,
      "playerName": "Kylian Mbappé",
      "fromClubName": "PSG",
      "toClubName": "Real Madrid",
      "feeEur": 180000000,
      "status": "CONFIRMED",
      "reliability": 95,
      "publishedAt": "2024-06-01T10:00:00",
      "journalistXHandle": "@FabrizioRomano",
      "journalistName": "Fabrizio Romano",
      "journalistCredibility": 98.5,
      "postContent": "Here we go! Kylian Mbappé to Real Madrid..."
    }
  ],
  "totalElements": 100,
  "totalPages": 5,
  "size": 20,
  "number": 0
}
```

---

## 기자 (Journalist)

### `GET /api/journalists`
기자 전체 목록 (공신력 점수 순, Redis TTL 30분 캐시)

**Response** `List<JournalistResponse>`

```json
[
  {
    "id": 1,
    "xHandle": "@FabrizioRomano",
    "name": "Fabrizio Romano",
    "profileImageUrl": "https://...",
    "followerCount": 24000000,
    "credibilityScore": 98.5,
    "rank": 1
  }
]
```

---

### `GET /api/journalists/{id}`
기자 상세 정보

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 기자 ID |

**Response** `JournalistResponse`

```json
{
  "id": 1,
  "xHandle": "@FabrizioRomano",
  "name": "Fabrizio Romano",
  "profileImageUrl": "https://...",
  "followerCount": 24000000,
  "credibilityScore": 98.5,
  "rank": 1
}
```

---

### `GET /api/journalists/{id}/news`
해당 기자가 최초 보도한 이적 뉴스 목록

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 기자 ID |

**Response** `List<TransferNewsResponse>`

---

## 선수 (Player)

### `GET /api/players/expiring`
계약 만료 임박 선수 목록 (contractUntil 오름차순)

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `months` | `int` | N | 만료 기준 기간 (기본값 6) |

**Response** `List<PlayerResponse>`

```json
[
  {
    "id": 7,
    "name": "Bukayo Saka",
    "nationality": "English",
    "position": "FW",
    "currentClubName": "Arsenal",
    "contractUntil": "2026-06-30",
    "contractStatus": "CONTRACTED",
    "profileImageUrl": "https://..."
  }
]
```

---

### `GET /api/players/{id}`
선수 상세 정보

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 선수 ID |

**Response** `PlayerResponse`

```json
{
  "id": 1,
  "name": "Kylian Mbappé",
  "nationality": "France",
  "position": "FW",
  "currentClubName": "Real Madrid",
  "contractUntil": "2029-06-30",
  "contractStatus": "CONTRACTED",
  "profileImageUrl": "https://..."
}
```

---

### `GET /api/players/{id}/transfers`
선수 이적 히스토리

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 선수 ID |

**Response** `List<TransferNewsResponse>`

---

## 구단 (Club)

### `GET /api/clubs/{id}`
구단 상세 정보

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 구단 ID |

**Response** `ClubResponse`

```json
{
  "id": 1,
  "name": "Real Madrid CF",
  "shortName": "Real Madrid",
  "logoUrl": "https://...",
  "city": "Madrid",
  "countryCode": "ES",
  "latitude": 40.4530,
  "longitude": -3.6883,
  "stadiumName": "Santiago Bernabéu",
  "leagueName": "La Liga"
}
```

---

### `GET /api/clubs/{id}/transfers`
구단 이적 인/아웃 목록

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 구단 ID |

**Response** `Map<String, List<TransferNewsResponse>>`

```json
{
  "incoming": [ /* TransferNewsResponse[] */ ],
  "outgoing": [ /* TransferNewsResponse[] */ ]
}
```

---

## 리그 (League)

### `GET /api/leagues`
5대 리그 목록 전체

**Response** `List<LeagueResponse>`

```json
[
  {
    "id": 1,
    "name": "Premier League",
    "countryCode": "EN",
    "logoUrl": "https://...",
    "tier": 1
  }
]
```

---

### `GET /api/leagues/{id}/clubs`
리그 소속 구단 목록 (지도 마커용 lat/lon 포함)

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 리그 ID |

**Response** `List<ClubResponse>`

---

### `GET /api/leagues/{id}/news`
리그별 이적 뉴스 (최신순, 페이지네이션)

**Path Parameters**

| 파라미터 | 타입 | 설명 |
|---------|------|------|
| `id` | `Long` | 리그 ID |

**Query Parameters**

| 파라미터 | 타입 | 필수 | 설명 |
|---------|------|------|------|
| `page` | `int` | N | 페이지 번호 (기본값 0) |
| `size` | `int` | N | 페이지 크기 (기본값 20) |

**Response** `Page<TransferNewsResponse>`

---

## 공통 응답 타입 참조

### `TransferNewsResponse`
| 필드 | 타입 | 설명 |
|-----|------|------|
| `id` | `Long` | 뉴스 ID |
| `playerName` | `String` | 선수 이름 |
| `fromClubName` | `String?` | 출발 구단 (null 가능) |
| `toClubName` | `String` | 도착 구단 |
| `feeEur` | `Long?` | 이적료 (유로) |
| `status` | `String` | `INTEREST` / `RUMOR` / `CONFIRMED` / `DENIED` / `LOAN` / `CONTRACT_EXTENSION` |
| `reliability` | `Byte` | 신뢰도 (0–100) |
| `publishedAt` | `LocalDateTime` | 최초 보도 시각 |
| `journalistXHandle` | `String` | 기자 X 핸들 |
| `journalistName` | `String` | 기자 이름 |
| `journalistCredibility` | `Float` | 기자 공신력 점수 |
| `postContent` | `String` | 원문 X 게시물 내용 |

### `JournalistResponse`
| 필드 | 타입 | 설명 |
|-----|------|------|
| `id` | `Long` | 기자 ID |
| `xHandle` | `String` | X 핸들 |
| `name` | `String` | 이름 |
| `profileImageUrl` | `String` | 프로필 이미지 |
| `followerCount` | `Integer` | 팔로워 수 |
| `credibilityScore` | `Float` | 공신력 점수 |
| `rank` | `Integer` | 공신력 순위 |

### `PlayerResponse`
| 필드 | 타입 | 설명 |
|-----|------|------|
| `id` | `Long` | 선수 ID |
| `name` | `String` | 이름 |
| `nationality` | `String` | 국적 |
| `position` | `String?` | 포지션 (`GK` / `DF` / `MF` / `FW`) |
| `currentClubName` | `String?` | 현재 소속팀 |
| `contractUntil` | `String?` | 계약 만료일 |
| `contractStatus` | `String?` | 계약 상태 (`FREE_AGENT` / `CONTRACTED` / `LOANED`) |
| `profileImageUrl` | `String` | 프로필 이미지 |

### `ClubResponse`
| 필드 | 타입 | 설명 |
|-----|------|------|
| `id` | `Long` | 구단 ID |
| `name` | `String` | 구단 명 |
| `shortName` | `String` | 약칭 |
| `logoUrl` | `String` | 로고 이미지 |
| `city` | `String` | 도시 |
| `countryCode` | `String` | 국가 코드 |
| `latitude` | `BigDecimal` | 위도 (지도 마커) |
| `longitude` | `BigDecimal` | 경도 (지도 마커) |
| `stadiumName` | `String` | 경기장 이름 |
| `leagueName` | `String?` | 소속 리그 |

### `LeagueResponse`
| 필드 | 타입 | 설명 |
|-----|------|------|
| `id` | `Long` | 리그 ID |
| `name` | `String` | 리그 명 |
| `countryCode` | `String` | 국가 코드 |
| `logoUrl` | `String` | 로고 이미지 |
| `tier` | `Integer` | 리그 등급 |
