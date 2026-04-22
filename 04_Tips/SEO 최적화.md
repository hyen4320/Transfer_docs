사용자가 `example.com`으로 들어오든 `www.example.com`으로 들어오든 한 곳으로 합쳐주는 것이 SEO(검색 엔진 최적화)와 관리에 좋습니다. 클라우드플레어의 **[규칙(Rules)] > [Page Rules]** 또는 **[Redirect Rules]**에서 리다이렉트 설정을 할 수 있습니다.

**설정 예시 (Redirect Rules):**

- **수신 요청:** `Hostname`이 `www.example.com`과 같을 때
    
- **URL 리다이렉션:** `https://example.com`으로 이동
    

이렇게 하면 `www`를 붙여서 들어오는 사람들도 깔끔하게 메인 주소로 연결됩니다. 혹시 지금 사용 중인 EC2 서버에 이미 웹 서버(Nginx나 Apache)를 올려두셨나요? 해당 설정에 따라 추가 작업이 필요할 수도 있습니다.
### 2단계: 리다이렉트 설정 (하나로 합치기)

사용자가 `www.mysite.com`으로 들어왔을 때 `mysite.com`으로 자동으로 보내주거나, 그 반대로 설정하는 과정입니다. 주소가 통일되어야 구글 검색 엔진(SEO) 최적화에 유리합니다.

클라우드플레어의 **[Rules(규칙)] > [Redirect Rules]** 메뉴를 이용하는 것이 가장 최신 방식입니다.

1. **[Create Rule]** 버튼을 누릅니다.
    
2. **규칙 이름:** `www to non-www` (자유롭게 입력)
    
3. **수신 요청(If):** 'Custom filter expression' 선택
    
    - **필드:** `Hostname`
        
    - **연산자:** `equals`
        
    - **값:** `www.내도메인.com`
        
4. **URL 리다이렉션(Then):**
    
    - **유형:** `Dynamic`
        
    - **대상 URL:** `concat("https://내도메인.com", http.request.uri.path)`
        
    - **상태 코드:** `301` (영구 이동)
        
5. **배포(Deploy)**를 누릅니다.
    

---

### 3단계: SSL/TLS 암호화 확인

`www`를 설정한 후 접속했을 때 "보안 연결이 되지 않음" 메시지가 뜨지 않게 하려면 SSL 설정을 확인해야 합니다.

- **[SSL/TLS] > [개요]** 메뉴에서 모드를 **전체(Full)** 또는 **전체(엄격)(Full - Strict)**로 설정하세요.
    
- **[SSL/TLS] > [Edge Certificates]** 메뉴에서 **항상 HTTPS 사용(Always Use HTTPS)** 기능을 켜두면, 사용자가 `http`로 들어와도 자동으로 보안 접속(`https`)으로 전환됩니다.