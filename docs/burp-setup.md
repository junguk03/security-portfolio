# 🔧 Burp Suite 설치 및 설정 가이드

> macOS 환경에서 Burp Suite Community Edition 설치 및 설정 방법

---

## 📥 1. 다운로드 및 설치

### 1.1 다운로드
1. https://portswigger.net/burp/releases 접속
2. **"Community Edition"** 선택 (무료)
3. **macOS** 버전 다운로드 (DMG 파일)

### 1.2 설치
```bash
# DMG 파일을 열고 Applications 폴더로 드래그
# 또는 터미널에서:
open ~/Downloads/burpsuite_community_macos_*.dmg
```

### 1.3 첫 실행
1. Launchpad에서 "Burp Suite Community" 실행
2. 라이선스 동의: **"I Accept"** 클릭
3. 프로젝트 선택: **"Temporary project"** 선택
4. 설정: **"Use Burp defaults"** 선택
5. **"Start Burp"** 클릭

---

## ⚙️ 2. 프록시 설정

### 2.1 Burp Proxy 확인
1. 상단 메뉴에서 **"Proxy"** 탭 클릭
2. **"Options"** 또는 **"Settings"** 클릭
3. **"Proxy Listeners"** 섹션에서 확인:
   - Address: `127.0.0.1:8080`
   - Running: ✅ (체크 상태)

### 2.2 Intercept 기능
- **Proxy > Intercept** 탭에서 **"Intercept is on"** 버튼 확인
- 처음에는 **"Intercept is off"**로 설정 (학습 초반)

---

## 🌐 3. 브라우저 프록시 설정

### 3.1 Firefox (추천)

**왜 Firefox?**
- 시스템 프록시가 아닌 자체 프록시 사용
- 다른 브라우저에 영향 없음
- FoxyProxy 확장 프로그램 지원

**설정 방법**:
1. Firefox 설정 열기 (⌘ + ,)
2. 검색창에 "프록시" 입력
3. **"네트워크 설정"** → **"설정"** 버튼 클릭
4. **"수동 프록시 설정"** 선택
5. 입력:
   - HTTP 프록시: `127.0.0.1`
   - 포트: `8080`
6. **"모든 프로토콜에 이 프록시 서버 사용"** 체크
7. **"확인"** 클릭

### 3.2 FoxyProxy 확장 프로그램 (선택)

**장점**: 프록시 on/off 빠른 전환

**설치**:
1. https://addons.mozilla.org/firefox/addon/foxyproxy-standard/
2. "Add to Firefox" 클릭
3. FoxyProxy 설정:
   - Proxy Type: HTTP
   - Proxy IP: `127.0.0.1`
   - Port: `8080`
   - Title: "Burp Suite"

---

## 🔒 4. CA 인증서 설치 (HTTPS용)

### 4.1 인증서 다운로드
1. Firefox 프록시 설정 완료 후
2. 브라우저에서 http://burpsuite 접속
3. **"CA Certificate"** 다운로드 (burp-ca-cert.der)

### 4.2 macOS 키체인에 설치
```bash
# 다운로드한 인증서 열기
open ~/Downloads/burp-ca-cert.der
```

1. "키체인 접근" 앱이 열림
2. 인증서를 "로그인" 또는 "시스템" 키체인에 추가
3. 추가된 인증서를 더블 클릭
4. **"신뢰"** 섹션 펼치기
5. **"이 인증서 사용 시"**: **"항상 신뢰"** 선택
6. 창 닫기 (관리자 비밀번호 입력 필요)

### 4.3 Firefox에 인증서 추가
1. Firefox 설정 (⌘ + ,)
2. **"개인정보 및 보안"**
3. **"인증서"** → **"인증서 보기"** 클릭
4. **"Authorities"** 탭
5. **"Import"** 클릭
6. `burp-ca-cert.der` 선택
7. **"Trust this CA to identify websites"** 체크
8. **"OK"** 클릭

---

## ✅ 5. 테스트

### 5.1 HTTP 트래픽 캡처 테스트
1. Burp Suite 실행 및 Proxy 탭 이동
2. **"Intercept is off"** 상태 확인
3. **"HTTP history"** 서브탭 열기
4. Firefox에서 http://example.com 접속
5. Burp Suite "HTTP history"에 요청 표시 확인 ✅

### 5.2 HTTPS 트래픽 캡처 테스트
1. Firefox에서 https://example.com 접속
2. Burp Suite "HTTP history"에 HTTPS 요청 표시 확인 ✅
3. 인증서 경고가 뜨면 → CA 인증서 설치 다시 확인

---

## 🎯 6. Burp Suite 기본 사용법

### 6.1 Proxy 탭
- **Intercept**: 요청/응답 가로채기 (수정 가능)
- **HTTP history**: 모든 HTTP/HTTPS 트래픽 기록
- **WebSockets history**: WebSocket 트래픽 기록

### 6.2 Target 탭
- **Site map**: 방문한 사이트 구조 표시
- **Scope**: 특정 도메인만 분석하도록 범위 설정

### 6.3 Repeater 탭
- HTTP 요청을 반복 전송 (파라미터 변경 테스트)
- 우클릭 → "Send to Repeater"로 추가

### 6.4 Intruder 탭
- 자동화된 공격 (Brute Force, Fuzzing 등)
- Community Edition은 속도 제한 있음

---

## 🔧 7. 자주 사용하는 기능

### 7.1 요청 수정하기
1. **Proxy > Intercept is on** 활성화
2. 브라우저에서 요청 발생
3. Burp에서 요청이 잡힘
4. 파라미터 수정 후 **"Forward"** 클릭

### 7.2 응답 수정하기
1. **Proxy > Options > Intercept Server Responses** 체크
2. 서버 응답을 가로채서 수정 가능

### 7.3 특정 도메인만 캡처
1. **Target > Scope** 탭
2. **"Add"** 클릭하여 도메인 추가
3. **Proxy > Options > "Use Scope"** 체크

---

## ⚠️ 8. 주의사항

### 사용 후 반드시 프록시 해제!
```bash
# Firefox 설정에서 프록시를 "시스템 프록시 설정 사용"으로 변경
# 또는 FoxyProxy를 "Direct" 모드로 전환
```

**이유**:
- Burp Suite 종료 후에도 프록시 설정이 남아있으면 인터넷 연결 안 됨
- 일상적인 브라우징이 느려짐

### 실습 환경과 실제 환경 분리
- 개인정보 입력하는 사이트는 프록시 끄고 사용
- 학습 전용 브라우저 프로필 생성 권장

---

## 📚 추가 학습 자료

- [Burp Suite 공식 문서](https://portswigger.net/burp/documentation)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [Burp Suite YouTube 튜토리얼](https://www.youtube.com/c/PortSwiggerTV)

---

**Last Updated**: 2025-12-14
**Burp Suite Version**: Community Edition (최신 버전)
