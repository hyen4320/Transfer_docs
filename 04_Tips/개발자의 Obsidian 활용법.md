# 개발자의 Obsidian 활용법

> 작성일: 2026-04-14

---

## 왜 개발자들이 Obsidian을 쓰는가?

- **로컬 Markdown 파일** — Git으로 버전 관리, 어느 에디터에서도 열림
- **백링크/그래프** — 기술 개념 간 연결 관계를 시각화
- **Dataview** — 노트를 코드처럼 쿼리해서 동적 대시보드 생성
- **Vim 키바인딩** 지원 — 에디터 벗어나지 않아도 됨
- **오프라인 우선** — 인터넷 없이도 완전히 동작

---

## 1. 개인 기술 위키 (Second Brain)

배운 것을 체계적으로 쌓아두는 개인 지식베이스.

**폴더 구조 예시:**
```
vault/
├── Languages/
│   ├── Java/
│   ├── Python/
│   └── TypeScript/
├── Frameworks/
│   ├── Spring Boot/
│   └── React/
├── CS/
│   ├── 자료구조/
│   ├── 네트워크/
│   └── OS/
├── Infra/
│   ├── Docker/
│   └── Kubernetes/
└── TIL/          ← Today I Learned
```

**노트 예시:**
```markdown
# Spring Security - JWT 인증 흐름

tags: #Spring #JWT #Security

## 흐름
1. 클라이언트 → 로그인 요청
2. 서버 → JWT 발급
3. 클라이언트 → 이후 요청마다 헤더에 토큰 첨부

## 관련 개념
- [[JwtTokenProvider]]
- [[SecurityFilterChain]]
- [[OAuth2]]

## 참고
- 공식 docs: ...
```

---

## 2. TIL (Today I Learned)

매일 배운 것을 짧게 기록하는 습관.

**템플릿 예시:**
```markdown
# TIL - {{date}}

## 오늘 배운 것

## 코드 스니펫

\`\`\`java

\`\`\`

## 참고 링크

## 내일 볼 것
```

- Calendar 플러그인으로 날짜별 TIL 달력 관리
- GitHub에 push해서 잔디 심기 겸 공개 TIL 운영하는 개발자도 많음

---

## 3. 프로젝트 문서화

코드 외에 설계 결정, 트러블슈팅 기록을 노트로 관리.

**구조 예시:**
```
Projects/Transfer/
├── README.md           ← 프로젝트 개요
├── dev-plan.md         ← 개발 계획
├── tech-stack.md       ← 기술 스택 선정 이유
├── erd.md              ← ERD 설명
├── ADR/                ← Architecture Decision Records
│   ├── 001-DB선택.md
│   └── 002-인증방식.md
└── Troubleshooting/    ← 문제 해결 기록
    ├── N+1-쿼리.md
    └── CORS-오류.md
```

**ADR (Architecture Decision Record) 템플릿:**
```markdown
# ADR-001: PostgreSQL 선택

## 상태
결정됨

## 컨텍스트
관계형 데이터가 많고 복잡한 조인 쿼리 필요

## 결정
PostgreSQL 사용

## 결과
- 장점: JSON 지원, 성능, 오픈소스
- 단점: MySQL 대비 운영 경험 부족
```

---

## 4. 코드 스니펫 관리

자주 쓰는 코드 패턴을 노트로 저장.

**예시 노트: `Spring - 공통 예외 처리.md`**
````markdown
# Spring - 공통 예외 처리

tags: #Spring #ExceptionHandler

\`\`\`java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(EntityNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(EntityNotFoundException e) {
        return ResponseEntity.status(404)
            .body(new ErrorResponse(e.getMessage()));
    }
}
\`\`\`

관련: [[ErrorResponse]], [[CustomException]]
````

- Dataview로 태그별 스니펫 목록 자동 생성 가능

---

## 5. 알고리즘/CS 문제 풀이 기록

```markdown
# LeetCode 42 - Trapping Rain Water

tags: #알고리즘 #TwoPointer #Hard

## 풀이 아이디어
양쪽 포인터에서 좁혀오며 낮은 쪽 기준으로 물 계산

## 코드

\`\`\`python
def trap(height):
    left, right = 0, len(height) - 1
    left_max = right_max = water = 0
    while left < right:
        if height[left] < height[right]:
            left_max = max(left_max, height[left])
            water += left_max - height[left]
            left += 1
        else:
            right_max = max(right_max, height[right])
            water += right_max - height[right]
            right -= 1
    return water
\`\`\`

## 시간복잡도
O(n)

## 비슷한 문제
- [[LeetCode 11 - Container With Most Water]]
```

---

## 6. 독서/강의 노트

기술 서적이나 강의를 들으며 핵심만 정리.

**구조 예시:**
```markdown
# 클린 코드 - 함수

## 핵심 원칙
- 함수는 한 가지만 해야 한다
- 함수 이름은 동사로 시작

## 인상 깊은 구절
> "함수는 작아야 한다. 더 작아야 한다."

## 적용할 것
- [ ] 서비스 레이어 메서드 분리 리팩토링

관련: [[단일 책임 원칙]], [[리팩토링]]
```

---

## 7. 그래프 뷰 활용

`Ctrl + G`로 노트 간 연결을 시각적으로 확인.

- 자주 연결되는 노트 = 핵심 개념 허브
- 고립된 노트 = 아직 연결 안 된 지식 → 백링크 추가 기회
- 클러스터링으로 지식 영역 파악

---

## 대중적인 vault 관리 방식

| 방식 | 설명 | 적합한 경우 |
|------|------|-------------|
| **PARA** | Projects/Areas/Resources/Archive | 업무+개인 통합 관리 |
| **Zettelkasten** | 원자적 노트 + 링크 연결 | 지식 축적, 글쓰기 |
| **Johnny Decimal** | 번호 기반 폴더 체계 | 파일 많고 체계 선호 |
| **TIL 중심** | 날짜별 짧은 기록 | 꾸준한 학습 기록 |

---

## GitHub 공개 vault 예시

많은 개발자들이 vault를 GitHub에 공개해 포트폴리오로 활용:
- `git init` → GitHub private/public repo 연결
- Git 플러그인으로 자동 커밋/푸시
- README에 주요 노트 링크 정리
