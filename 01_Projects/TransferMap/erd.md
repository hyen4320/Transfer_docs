---
tags: [transfermap, design, db]
type: design
updated: 2026-04-26
---

# E-R Diagram

## Concept
해외축구 이적시장 소식을 X(트위터)의 고신뢰 기자 게시물로 수집하고,
기사 등록 속도·정확도·파급력 기반으로 기자 공신력을 산출하여 유럽 지도 UI에 제공하는 서비스.

---

## 원천 데이터

| 출처 | 설명 | 대상 테이블 |
|------|------|-------------|
| **X API v2** | 기자 게시물 15분 주기 수집 (userId 기반) | JOURNALIST, POST, TRANSFER_NEWS |
| **Kaggle Transfermarkt** | 2000/01~2025/26 시즌 5대 리그 이적 이력 | TRANSFER_NEWS, PLAYER, CLUB |
| **수동 입력** | 구단 좌표·경기장·리그 메타 | CLUB, LEAGUE, CLUB_SEASON |

---

```mermaid
erDiagram

    JOURNALIST {
        BIGINT      journalist_id       PK
        VARCHAR     x_handle            UK  "X(Twitter) 계정명"
        VARCHAR     name
        VARCHAR     profile_image_url
        INT         follower_count
        FLOAT       credibility_score       "종합 공신력 점수 (0~100)"
        INT         rank                    "공신력 기반 순위"
        TIMESTAMP   last_synced_at
        TIMESTAMP   created_at
    }

    POST {
        BIGINT      post_id             PK
        BIGINT      journalist_id       FK
        VARCHAR     x_post_id           UK  "원본 X 게시물 ID"
        TEXT        content
        INT         like_count
        INT         retweet_count
        INT         reply_count
        INT         view_count
        TIMESTAMP   posted_at
        TIMESTAMP   collected_at
    }

    TRANSFER_NEWS {
        BIGINT      news_id             PK
        BIGINT      post_id             FK
        BIGINT      player_id           FK
        BIGINT      from_club_id        FK  "이적 출발 구단 (null = 자유계약)"
        BIGINT      to_club_id          FK  "이적 도착 구단"
        BIGINT      fee_eur                 "이적료 (유로, null = 불명)"
        VARCHAR     status                  "INTEREST | RUMOR | CONFIRMED | DENIED | LOAN | CONTRACT_EXTENSION"
        SMALLINT    season                  "시즌 인코딩: 앞두자리+뒷두자리 (예: 25/26 → 51)"
        VARCHAR     transfer_window         "SUMMER | WINTER"
        TIMESTAMP   published_at
    }

    VERIFICATION {
        BIGINT      verification_id     PK
        BIGINT      news_id             FK
        BOOLEAN     is_confirmed
        TIMESTAMP   confirmed_at
        VARCHAR     source_url              "공식 발표 URL"
        BIGINT      confirmed_by        FK  "확정 뉴스를 올린 journalist_id"
    }

    CREDIBILITY_METRIC {
        BIGINT      metric_id           PK
        BIGINT      journalist_id       FK
        FLOAT       speed_score             "최초 보도까지 걸린 시간 역수 환산"
        FLOAT       accuracy_score          "검증된 뉴스 / 전체 뉴스 비율"
        FLOAT       impact_score            "RT·조회수·팔로워 기반 파급력"
        DATE        measured_date           "지표 산출 기준일 (월별 스냅샷)"
        SMALLINT    season
        VARCHAR     transfer_window
        INT         rank                    "해당 시즌+윈도우 기준 공신력 순위"
        TIMESTAMP   created_at
    }

    PLAYER {
        BIGINT      player_id           PK
        VARCHAR     transfermarkt_id    UK  "Transfermarkt 선수 ID (Kaggle 매핑키)"
        VARCHAR     name
        VARCHAR     nationality
        VARCHAR     position                "GK | DF | MF | FW"
        BIGINT      current_club_id     FK
        DATE        contract_until
        VARCHAR     profile_image_url
        VARCHAR     contract_status         "FREE_AGENT | CONTRACTED | LOANED"
    }

    CLUB {
        BIGINT      club_id             PK
        BIGINT      league_id           FK
        BIGINT      kaggle_club_id      UK  "Kaggle transfers.csv club_id (이적 매핑키)"
        VARCHAR     name
        VARCHAR     short_name
        VARCHAR     logo_url
        VARCHAR     city
        VARCHAR     country_code            "ISO 3166-1 alpha-2"
        DECIMAL     latitude                "지도 UI 마커용"
        DECIMAL     longitude
        VARCHAR     stadium_name
    }

    CLUB_ALIASES {
        BIGINT      id                  PK
        BIGINT      club_id             FK
        VARCHAR     alias               UK  "단축명·별칭 (대소문자 구분 없이 유니크)"
        VARCHAR     lang                    "언어코드 (기본 en)"
        VARCHAR     source                  "manual | transfermarkt | wikipedia"
    }

    CLUB_SEASON {
        BIGINT      club_season_id      PK
        BIGINT      club_id             FK
        SMALLINT    season                  "시즌 인코딩"
        BIGINT      league_id           FK
    }

    LEAGUE {
        BIGINT      league_id           PK
        VARCHAR     name
        VARCHAR     country_code
        VARCHAR     logo_url
        INT         tier                    "1부·2부 리그 구분"
    }

    NOTICE {
        BIGINT      notice_id           PK
        VARCHAR     tag                     "UPDATE | NOTICE | MAINTENANCE"
        VARCHAR     title
        TEXT        body
        DATE        published_at
    }

    %% ── Relationships ──────────────────────────────────────

    JOURNALIST      ||--o{    POST                : "작성"
    POST            ||--o|    TRANSFER_NEWS       : "추출"
    TRANSFER_NEWS   ||--o|    VERIFICATION        : "검증"
    JOURNALIST      ||--o{    CREDIBILITY_METRIC  : "측정"

    VERIFICATION    }o--||    JOURNALIST          : "confirmed_by"

    PLAYER          ||--o{    TRANSFER_NEWS       : "대상 선수"
    CLUB            ||--o{    TRANSFER_NEWS       : "from_club"
    CLUB            ||--o{    TRANSFER_NEWS       : "to_club"
    CLUB            }o--||    LEAGUE              : "소속"
    PLAYER          }o--||    CLUB                : "현 소속"
    CLUB            ||--o{    CLUB_ALIASES        : "별칭"
    CLUB            ||--o{    CLUB_SEASON         : "시즌 참가"
    LEAGUE          ||--o{    CLUB_SEASON         : "리그"
```

---

## 핵심 비즈니스 규칙

| 규칙 | 설명 |
|------|------|
| 공신력 점수 산출 | `credibility_score = speed_score × 0.3 + accuracy_score × 0.5 + impact_score × 0.2` |
| 정확도 측정 기준 | `status = CONFIRMED` & `VERIFICATION.is_confirmed = true` 건수 / 전체 뉴스 건수 |
| 속도 점수 | 동일 선수 이적 루머 중 `POST.posted_at` 최초 기자에게 높은 점수 |
| 파급력 점수 | `(retweet_count × 3 + like_count + view_count × 0.1) / follower_count` 정규화 |
| 지도 UI | `CLUB.latitude / longitude` 기준 마커, `to_club_id` 기준 이동 경로 시각화 |
| 자유계약 이적 | `TRANSFER_NEWS.from_club_id = NULL`, `fee_eur = 0` |
| 시즌 인코딩 | `season = 앞두자리 + 뒷두자리` — 25/26 → 51, 24/25 → 49. 홀수 패턴, 2씩 증가 |
| 이적 윈도우 | `SUMMER`: 6~9월 / `WINTER`: 1~2월. `published_at` 월 기준 자동 분류 |
| Kaggle 구단 매핑 | `CLUB.kaggle_club_id` = `transfers.csv`의 `from_club_id` / `to_club_id`. 직접 정수 매핑으로 이름 매칭 오류 방지 |
| 선수 중복 방지 | `PLAYER.transfermarkt_id` (UK) 기준으로 `WHERE NOT EXISTS` 패턴으로 삽입 |
| CLUB_SEASON 의미 | 특정 시즌에 해당 리그에 소속된 구단 기록. 강등/승격 이력 추적 가능 |
| 시즌별 기자 랭킹 | `CREDIBILITY_METRIC.rank` — 특정 `season + window` 조합 기준 순위 |
