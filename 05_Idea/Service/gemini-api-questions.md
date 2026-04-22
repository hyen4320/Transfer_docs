---
tags:
  - dev
  - gemini
  - llm
date: 2026-04-22
---
# Gemini API 질문 — 비용 최적화 & tool_use 패턴

현재 구현: `GeminiApiClient.parseTransferTweet()` — 트윗 1건당 API 1회 호출, `functionDeclarations` + `ANY` mode 사용.

---

## 비용 최적화

### Q1. 모델 선택
- `gemini-2.0-flash` vs `gemini-2.0-flash-lite` — 이적 트윗 파싱 정확도에 실질적 차이가 있는가?
- 단순 구조화 추출(playerName, fromClub, toClub, status)이면 Lite로 충분한가?

### Q2. 배치 처리
- 트윗 여러 건을 하나의 요청으로 묶을 수 있는가? (배열로 넘기고 배열로 받기)
- 묶을 경우 `functionCall` 응답이 배열로 오는가, 아니면 별도 처리가 필요한가?
- 현재 스케줄러가 15분마다 실행 — 묶을 때 최적 배치 사이즈는?

### Q3. Context Caching
- system instruction(함수 정의 + 파싱 규칙)을 캐싱할 수 있는가?
- `cachedContent` API를 쓰면 `tools`/`functionDeclarations` 반복 전송 비용을 줄일 수 있는가?
- TTL 최소값과 과금 방식은?

### Q4. 토큰 절감
- 현재 프롬프트: `"Extract football transfer information from this tweet: " + tweetContent`
- system instruction으로 분리하면 캐싱 적용 가능한가?
- 트윗 전처리(해시태그·멘션 제거)가 토큰 절감에 의미 있는 수준인가?

---

## tool_use (Function Calling) 패턴

### Q5. `ANY` vs `AUTO` mode
- 현재 `mode: ANY` + `allowedFunctionNames: ["extract_transfer"]` — 함수를 항상 호출하도록 강제.
- `AUTO`로 바꾸면 "이적 뉴스 아님"을 함수 호출 없이 처리할 수 있는가?
- `ANY`가 `isTransferNews: false` 케이스에서 불필요한 토큰을 쓰는가?

### Q6. 응답 스키마 vs Function Calling
- `responseSchema` (JSON mode)와 `functionDeclarations` 중 구조화 추출에 더 적합한 것은?
- `responseSchema`가 더 싸거나 빠른가?

### Q7. 다중 엔티티 처리
- 트윗 1건에 여러 선수가 언급될 때 (예: "A to B, C to D") — 현재 구조에서 배열 반환이 가능한가?
- `parameters.type: array`로 선언하면 되는가?

### Q8. 에러 & 재시도
- `RESOURCE_EXHAUSTED` (rate limit) 발생 시 권장 retry 전략은? (exponential backoff 기본값)
- `finishReason: "SAFETY"` 로 막힐 가능성이 있는가? (이적 루머 트윗)

---

## 현재 구현 메모

```
모델: props.getModel() (application.properties에서 주입)
호출 방식: RestClient POST /v1beta/models/{model}:generateContent?key={apiKey}
토큰 로깅: usageMetadata.promptTokenCount / candidatesTokenCount (DEBUG)
실패 처리: 예외 발생 시 ParseResult.notTransfer() 반환
```

비용 확인: [Google AI Studio — Usage](https://aistudio.google.com/app/usage)
