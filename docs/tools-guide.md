# 🔧 보안 도구 사용 가이드

> 보안 학습 및 테스트에 사용하는 도구들의 설치 및 사용법

---

## 📚 도구 목록

### 필수 도구
1. [Burp Suite Community](#1-burp-suite-community) - 웹 프록시 및 보안 테스트
2. [curl](#2-curl) - HTTP 요청 테스트
3. [jq](#3-jq) - JSON 데이터 파싱

### 추천 도구
4. [jwt_tool](#4-jwt_tool) - JWT 분석 및 공격
5. [ffuf](#5-ffuf) - 웹 퍼징
6. [sqlmap](#6-sqlmap) - SQL Injection 자동화

### 선택 도구
7. [Postman](#7-postman) - API 테스트
8. [Docker](#8-docker) - 격리된 환경 구성

---

## 1. Burp Suite Community

### 설치
[burp-setup.md](burp-setup.md) 참고

### 기본 사용법

#### 1.1 HTTP 요청 가로채기
```
1. Burp Suite 실행
2. Proxy > Intercept is on 활성화
3. 브라우저에서 요청 발생
4. Burp에서 요청 확인 및 수정
5. Forward 클릭하여 전송
```

#### 1.2 Repeater로 요청 반복
```
1. HTTP history에서 요청 우클릭
2. "Send to Repeater" 선택
3. Repeater 탭에서 파라미터 수정
4. "Send" 클릭하여 반복 전송
```

#### 1.3 Intruder로 자동화
```
1. 요청 우클릭 → "Send to Intruder"
2. Positions 탭에서 공격 지점 설정
3. Payloads 탭에서 페이로드 설정
4. "Start attack" 클릭
```

---

## 2. curl

### 설치
```bash
# macOS (기본 설치되어 있음)
curl --version

# 최신 버전 설치
brew install curl
```

### 기본 사용법

#### 2.1 GET 요청
```bash
# 기본 GET
curl https://api.example.com/users

# 헤더 포함
curl -H "Authorization: Bearer TOKEN" https://api.example.com/users

# 상세 정보 출력 (-v: verbose)
curl -v https://api.example.com/users
```

#### 2.2 POST 요청
```bash
# JSON 데이터 전송
curl -X POST https://api.example.com/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"test"}'

# Form 데이터 전송
curl -X POST https://api.example.com/login \
  -d "username=admin&password=test"
```

#### 2.3 쿠키 사용
```bash
# 쿠키 저장
curl -c cookies.txt https://api.example.com/login

# 쿠키 전송
curl -b cookies.txt https://api.example.com/profile
```

#### 2.4 파일 다운로드
```bash
# 파일 다운로드 (-O: 원본 파일명, -o: 지정 파일명)
curl -O https://example.com/file.pdf
curl -o myfile.pdf https://example.com/file.pdf
```

---

## 3. jq

### 설치
```bash
# macOS
brew install jq
```

### 기본 사용법

#### 3.1 JSON 포맷팅
```bash
# API 응답을 jq로 파싱
curl https://api.example.com/users | jq

# 파일에서 읽기
cat response.json | jq
```

#### 3.2 데이터 추출
```bash
# 특정 필드 추출
curl https://api.example.com/users | jq '.data.users'

# 배열 첫 번째 요소
curl https://api.example.com/users | jq '.[0]'

# 특정 필드만 선택
curl https://api.example.com/users | jq '.[] | {id, name}'
```

#### 3.3 필터링
```bash
# 조건 필터링
jq '.[] | select(.age > 20)' users.json

# 배열 길이
jq 'length' users.json
```

---

## 4. jwt_tool

### 설치
```bash
# Python 3 필요
git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool
pip3 install -r requirements.txt
```

### 기본 사용법

#### 4.1 JWT 디코딩
```bash
python3 jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### 4.2 JWT Crack (Brute Force)
```bash
# 약한 비밀키 찾기
python3 jwt_tool.py <JWT> -C -d /path/to/wordlist.txt

# 빠른 크랙 (짧은 비밀키)
python3 jwt_tool.py <JWT> -C -d common_secrets.txt
```

#### 4.3 alg: none 공격
```bash
python3 jwt_tool.py <JWT> -X a
```

#### 4.4 서명 변조
```bash
# 페이로드 변경 후 재서명
python3 jwt_tool.py <JWT> -T
```

---

## 5. ffuf

### 설치
```bash
# macOS
brew install ffuf
```

### 기본 사용법

#### 5.1 디렉토리 브루트포스
```bash
# 디렉토리 찾기
ffuf -u https://example.com/FUZZ -w wordlist.txt

# 상태 코드 필터링
ffuf -u https://example.com/FUZZ -w wordlist.txt -mc 200,301,302
```

#### 5.2 파라미터 퍼징
```bash
# GET 파라미터 퍼징
ffuf -u "https://example.com/api?id=FUZZ" -w numbers.txt

# POST 데이터 퍼징
ffuf -u https://example.com/login -X POST \
  -d "username=admin&password=FUZZ" -w passwords.txt
```

#### 5.3 서브도메인 열거
```bash
ffuf -u https://FUZZ.example.com -w subdomains.txt
```

---

## 6. sqlmap

### 설치
```bash
# macOS
brew install sqlmap
```

### 기본 사용법

#### 6.1 자동 SQL Injection 테스트
```bash
# GET 파라미터 테스트
sqlmap -u "https://example.com/user?id=1"

# POST 데이터 테스트
sqlmap -u https://example.com/login --data="username=admin&password=test"
```

#### 6.2 쿠키 포함
```bash
sqlmap -u "https://example.com/profile" --cookie="session=abc123"
```

#### 6.3 데이터베이스 덤프
```bash
# 데이터베이스 목록
sqlmap -u "https://example.com/user?id=1" --dbs

# 테이블 목록
sqlmap -u "https://example.com/user?id=1" -D dbname --tables

# 데이터 덤프
sqlmap -u "https://example.com/user?id=1" -D dbname -T users --dump
```

---

## 7. Postman

### 설치
```bash
# macOS
brew install --cask postman

# 또는 웹사이트에서 다운로드
# https://www.postman.com/downloads/
```

### 기본 사용법

#### 7.1 요청 만들기
```
1. "New" → "HTTP Request" 클릭
2. 메서드 선택 (GET, POST 등)
3. URL 입력
4. Headers 탭에서 헤더 추가
5. Body 탭에서 데이터 입력
6. "Send" 클릭
```

#### 7.2 Collection 관리
```
1. "New" → "Collection" 클릭
2. 요청들을 Collection에 저장
3. 환경 변수 설정 (예: {{base_url}})
```

#### 7.3 테스트 자동화
```javascript
// Tests 탭에서 JavaScript 작성
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response contains token", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.token).to.exist;
});
```

---

## 8. Docker

### 설치
```bash
# macOS
brew install --cask docker

# 또는 웹사이트에서 다운로드
# https://www.docker.com/products/docker-desktop
```

### 기본 사용법

#### 8.1 취약한 웹 앱 실행 (DVWA)
```bash
# DVWA (Damn Vulnerable Web Application) 실행
docker run -d -p 80:80 vulnerables/web-dvwa

# 브라우저에서 http://localhost 접속
# 기본 계정: admin / password
```

#### 8.2 WebGoat 실행
```bash
# WebGoat (OWASP 교육용 앱) 실행
docker run -d -p 8080:8080 -p 9090:9090 webgoat/goatandwolf

# 브라우저에서 http://localhost:8080/WebGoat 접속
```

#### 8.3 컨테이너 관리
```bash
# 실행 중인 컨테이너 확인
docker ps

# 컨테이너 중지
docker stop <container_id>

# 컨테이너 삭제
docker rm <container_id>
```

---

## 🎯 학습용 취약한 앱

### 1. DVWA (Damn Vulnerable Web Application)
- **설치**: `docker run -p 80:80 vulnerables/web-dvwa`
- **학습 내용**: SQL Injection, XSS, CSRF 등

### 2. WebGoat (OWASP)
- **설치**: `docker run -p 8080:8080 webgoat/goatandwolf`
- **학습 내용**: OWASP Top 10 실습

### 3. Juice Shop (OWASP)
- **설치**: `docker run -p 3000:3000 bkimminich/juice-shop`
- **학습 내용**: 현대적인 웹 취약점

---

## 📝 유용한 명령어 모음

### Burp Suite + curl 연계
```bash
# Burp Proxy를 통해 curl 요청 전송
curl --proxy http://127.0.0.1:8080 https://example.com
```

### JWT 디코딩 (jq 사용)
```bash
# JWT Payload 디코딩
echo "eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIn0" | base64 -d | jq
```

### SQL Injection 테스트 (curl)
```bash
# 간단한 SQL Injection 테스트
curl "https://example.com/user?id=1' OR '1'='1"
```

---

## 📚 추가 학습 자료

- [Burp Suite Documentation](https://portswigger.net/burp/documentation)
- [curl Tutorial](https://curl.se/docs/manual.html)
- [jq Tutorial](https://stedolan.github.io/jq/tutorial/)
- [ffuf GitHub](https://github.com/ffuf/ffuf)
- [sqlmap Documentation](https://github.com/sqlmapproject/sqlmap/wiki)

---

**Last Updated**: 2025-12-14
