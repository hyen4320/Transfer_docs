# Google AdSense

## 신청 전 준비사항

### 사이트 요건
- 콘텐츠가 충분해야 함 (최소 20~30개 이상 글 권장)
- 자체 도메인 필수 (blogspot, github.io 등 서브도메인은 불리)
- 개인정보처리방침 페이지 필수
- 문의(Contact) 페이지 권장
- About 페이지 권장
- 저작권 위반 콘텐츠, 성인/도박/폭력 콘텐츠 없어야 함

### 트래픽 관련
- 공식 최소 트래픽 기준은 없으나, 실제로는 어느 정도 유입이 있는 상태에서 승인율이 높음
- 구글 검색 노출이 되고 있어야 유리 (Search Console 연동 후 인덱싱 확인)

---

## 신청 절차

1. [https://adsense.google.com](https://adsense.google.com) 접속
2. 구글 계정으로 로그인
3. 사이트 URL 입력 및 국가 선택
4. AdSense 코드(스니펫)를 사이트 `<head>` 태그 안에 삽입
5. 검토 요청 제출
6. 구글 검토 기간: 보통 수일 ~ 수 주 소요
7. 승인 완료 시 이메일 수신 → 광고 단위 생성 후 게재

---

## 코드 삽입 방법

### HTML 직접 삽입
```html
<head>
  <!-- AdSense 자동 광고 코드 -->
  <script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXXXXXXXXXXXXXX"
       crossorigin="anonymous"></script>
</head>
```

### React (Vite) 프로젝트
`index.html`의 `<head>` 안에 삽입:
```html
<script async
  src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-XXXXXXXXXXXXXXXX"
  crossorigin="anonymous">
</script>
```

광고 단위를 컴포넌트로 사용할 경우:
```tsx
import { useEffect } from 'react';

export default function AdBanner() {
  useEffect(() => {
    try {
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    } catch (e) {}
  }, []);

  return (
    <ins
      className="adsbygoogle"
      style={{ display: 'block' }}
      data-ad-client="ca-pub-XXXXXXXXXXXXXXXX"
      data-ad-slot="XXXXXXXXXX"
      data-ad-format="auto"
      data-full-width-responsive="true"
    />
  );
}
```

---

## 광고 종류

| 유형 | 설명 |
|------|------|
| 자동 광고 | 구글이 최적 위치에 자동 배치 — 설정 간단 |
| 디스플레이 광고 | 직접 위치 지정, 배너 형태 |
| 인피드 광고 | 목록/피드 사이에 삽입 |
| 콘텐츠 내 광고 | 본문 중간에 삽입 |
| 검색 광고 | 사이트 내 검색 결과에 표시 |

---

## 주의사항 및 팁

### 승인 관련
- 신청 후 코드를 삽입하지 않으면 구글이 사이트를 크롤링할 수 없으므로 코드는 반드시 유지
- 승인 전에도 코드는 계속 `<head>`에 있어야 함 (광고는 승인 후 노출)
- 재심사 신청은 심사 결과 이메일의 안내를 따름

### 정책 위반 금지
- 자신의 광고 클릭 절대 금지 (계정 정지)
- 클릭 유도 문구 금지 ("광고를 클릭해주세요" 등)
- 팝업/팝언더로 광고 덮기 금지
- 광고와 일반 콘텐츠를 혼동하게 만드는 레이아웃 금지

### 수익 최적화
- 자동 광고를 먼저 켜두고 데이터 쌓은 뒤 수동 최적화
- 광고 밀도는 콘텐츠 대비 과하지 않게 (구글 가이드: 광고가 콘텐츠보다 많으면 안 됨)
- 모바일 반응형 광고 단위 사용 (`data-full-width-responsive="true"`)
- 스크롤 상단/본문 중간/본문 하단 조합이 일반적으로 효과적

### 개인정보처리방침
AdSense를 사용하면 사이트 개인정보처리방침에 다음 내용을 포함해야 함:
- 제3자(Google)가 쿠키를 사용해 광고를 제공함
- 사용자가 [Google 광고 설정](https://www.google.com/settings/ads)에서 개인화 광고를 거부할 수 있음

---

## 유용한 링크

- AdSense 정책 센터: https://support.google.com/adsense/answer/48182
- AdSense 도움말: https://support.google.com/adsense
- Google Search Console: https://search.google.com/search-console
