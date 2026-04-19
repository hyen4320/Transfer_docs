---
tags: [transfermap, design, db]
type: design
updated: 2026-04-16
---

# E-R Diagram

## Concept
해외축구 이적시장 소식을 X(트위터)의 고신뢰 기자 게시물로 수집하고,
기사 등록 속도·정확도·파급력 기반으로 기자 공신력을 산출하여 유럽 지도 UI에 제공하는 서비스.

---

```mermaid
erDiagram

    JOURNALIST {
        BIGINT      journalist_id   PK
        VARCHAR     x_handle        UK "X(Twitter) 계정명"
        VARCHAR     name
        VARCHAR     profile_image_url
        INT         follower_count
        FLOAT       credibility_score   "종합 공신력 점수 (0~100)"
        INT         rank                "공신력 기반 순위"
        TIMESTAMP   last_synced_at
        TIMESTAMP   created_at
    }

    POST {
        BIGINT      post_id         PK
        BIGINT      journalist_id   FK
        VARCHAR     x_post_id       UK "원본 X 게시물 ID"
        TEXT        content
        INT         like_count
        INT         retweet_count
        INT         reply_count
        INT         view_count
        TIMESTAMP   posted_at
        TIMESTAMP   collected_at
    }

    TRANSFER_NEWS {
        BIGINT      news_id         PK
        BIGINT      post_id         FK
        BIGINT      player_id       FK
        BIGINT      from_club_id    FK  "이적 출발 구단 (null = 자유계약)"
        BIGINT      to_club_id      FK  "이적 도착 구단"
        BIGINT      fee_eur             "이적료 (유로, null = 불명)"
        VARCHAR     status              "RUMOR | CONFIRMED | DENIED | LOAN"
        TINYINT     reliability         "1~5 : 기사 자체 신뢰도 레이블"
        SMALLINT    season              "시즌 인코딩: 앞두자리+뒷두자리 (예: 24/25 → 49)"
        VARCHAR     transfer_window     "이적 윈도우: SUMMER | WINTER (예약어 회피)"
        TIMESTAMP   published_at
    }

    VERIFICATION {
        BIGINT      verification_id PK
        BIGINT      news_id         FK
        BOOLEAN     is_confirmed
        TIMESTAMP   confirmed_at
        VARCHAR     source_url          "공식 발표 URL"
        BIGINT      confirmed_by        FK  "확정 뉴스를 올린 journalist_id"
    }

    CREDIBILITY_METRIC {
        BIGINT      metric_id       PK
        BIGINT      journalist_id   FK
        FLOAT       speed_score         "최초 보도까지 걸린 시간 역수 환산"
        FLOAT       accuracy_score      "검증된 뉴스 / 전체 뉴스 비율"
        FLOAT       impact_score        "RT·조회수·팔로워 기반 파급력"
        DATE        measured_date       "지표 산출 기준일 (월별 스냅샷)"
        SMALLINT    season              "시즌 인코딩: 앞두자리+뒷두자리 (예: 24/25 → 49)"
        VARCHAR     transfer_window     "이적 윈도우: SUMMER | WINTER (예약어 회피)"
        INT         rank                "해당 시즌+윈도우 기준 공신력 순위"
        TIMESTAMP   created_at
    }

    PLAYER {
        BIGINT      player_id       PK
        VARCHAR     name
        VARCHAR     nationality
        VARCHAR     position            "GK | DF | MF | FW"
        BIGINT      current_club_id FK
        DATE        contract_until
        VARCHAR     profile_image_url
    }

    CLUB {
        BIGINT      club_id         PK
        BIGINT      league_id       FK
        VARCHAR     name
        VARCHAR     short_name
        VARCHAR     logo_url
        VARCHAR     city
        VARCHAR     country_code        "ISO 3166-1 alpha-2"
        DECIMAL     latitude            "지도 UI 마커용"
        DECIMAL     longitude
        VARCHAR     stadium_name
    }

    LEAGUE {
        BIGINT      league_id       PK
        VARCHAR     name
        VARCHAR     country_code
        VARCHAR     logo_url
        INT         tier                "1부·2부 리그 구분"
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
```

---

## 핵심 비즈니스 규칙

| 규칙 | 설명 |
|------|------|
| 공신력 점수 산출 | `credibility_score = speed_score × 0.3 + accuracy_score × 0.5 + impact_score × 0.2` (가중치는 조정 가능) |
| 정확도 측정 기준 | `TRANSFER_NEWS.status`가 `CONFIRMED`이고 `VERIFICATION.is_confirmed = true`인 건수 / 전체 뉴스 건수 |
| 속도 점수 | 동일 선수의 이적 루머 중 가장 먼저 `POST.posted_at`이 기록된 기자에게 높은 점수 부여 |
| 파급력 점수 | `(retweet_count × 3 + like_count + view_count × 0.1) / follower_count` 정규화 |
| 지도 UI | `CLUB.latitude` / `CLUB.longitude` 기준으로 유럽 지도 마커 렌더링, `to_club_id` 기준 이동 경로 시각화 |
| 자유계약 이적 | `TRANSFER_NEWS.from_club_id = NULL`, `fee_eur = 0` |
| 시즌 인코딩 | `season = 앞두자리 + 뒷두자리` — 24/25 → 49, 25/26 → 51. 2씩 증가하는 홀수 패턴으로 고유값 보장. 현재 시즌 조회 시 `WHERE season = 49` |
| 이적 윈도우 | `SUMMER`: 6~9월 (시즌 시작 전 여름), `WINTER`: 1~2월 (시즌 중반 겨울). 동일 시즌(49)에 두 윈도우가 모두 존재. `published_at` 월 기준으로 자동 분류 가능 |
| 시즌별 기자 랭킹 | `CREDIBILITY_METRIC.rank` — 특정 `season + window` 조합 기준 순위. `JOURNALIST.rank`는 가장 최근 스냅샷 캐시값 |
```
