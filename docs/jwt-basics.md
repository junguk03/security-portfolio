# 🔑 JWT (JSON Web Token) 기초

> JWT의 구조, 동작 원리, 보안 취약점 및 방어 방법

---

## 📖 1. JWT란?

**JSON Web Token (JWT)**는 JSON 객체를 사용하여 정보를 안전하게 전송하기 위한 토큰 기반 인증 표준입니다.

### 특징
- **Self-contained**: 토큰 자체에 사용자 정보 포함
- **Stateless**: 서버에 세션 저장 불필요
- **URL-safe**: Base64 인코딩으로 URL에 사용 가능

### 사용 사례
- **인증 (Authentication)**: 로그인 후 사용자 신원 확인
- **정보 교환 (Information Exchange)**: 서명된 데이터 전송

---

## 🏗️ 2. JWT 구조

JWT는 `.`으로 구분된 3개의 부분으로 구성:

```
Header.Payload.Signature
```

### 예시
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

### 2.1 Header (헤더)

토큰의 타입과 해싱 알고리즘 정의

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- **alg**: 서명 알고리즘 (HS256, RS256 등)
- **typ**: 토큰 타입 (JWT)

**Base64 인코딩**: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9`

---

### 2.2 Payload (페이로드)

사용자 정보 및 클레임(Claim) 포함

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022,
  "exp": 1516242622,
  "role": "admin"
}
```

#### 클레임 종류
- **Registered Claims** (등록된 클레임):
  - `iss` (Issuer): 발급자
  - `sub` (Subject): 주제 (사용자 ID)
  - `aud` (Audience): 대상자
  - `exp` (Expiration Time): 만료 시간
  - `iat` (Issued At): 발급 시간

- **Public Claims** (공개 클레임):
  - 충돌 방지를 위해 URI 형식 권장

- **Private Claims** (비공개 클레임):
  - 커스텀 정보 (name, role 등)

**Base64 인코딩**: `eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ`

---

### 2.3 Signature (서명)

Header + Payload를 비밀키로 서명

```javascript
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**서명 값**: `SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c`

---

## 🔐 3. JWT 동작 원리

### 3.1 인증 플로우

```
1. 사용자 로그인 요청
   POST /login { username, password }

2. 서버가 JWT 생성
   - Header + Payload 생성
   - 비밀키로 서명
   - JWT 반환

3. 클라이언트가 JWT 저장
   - localStorage, sessionStorage, Cookie 등

4. 이후 요청마다 JWT 전송
   GET /api/user
   Authorization: Bearer eyJhbGci...

5. 서버가 JWT 검증
   - 서명 확인
   - 만료 시간 확인
   - 사용자 인증 완료
```

---

## ⚠️ 4. JWT 보안 취약점

### 4.1 alg: none 공격

**취약점**: 알고리즘을 "none"으로 변경하여 서명 우회

```json
// Header
{
  "alg": "none",
  "typ": "JWT"
}
```

**공격 예시**:
```
eyJhbGciOiJub25lIiwidHlwIjoiSldUIn0.eyJzdWIiOiJhZG1pbiJ9.
```

**방어**:
```typescript
// ❌ 취약한 코드
if (header.alg === "none") {
  // 서명 검증 생략
}

// ✅ 안전한 코드
const allowedAlgorithms = ["HS256", "RS256"];
if (!allowedAlgorithms.includes(header.alg)) {
  throw new Error("Invalid algorithm");
}
```

---

### 4.2 Weak Secret (약한 비밀키)

**취약점**: 짧거나 예측 가능한 비밀키 사용

```typescript
// ❌ 취약한 코드
const secret = "secret";  // 너무 짧음
const secret = "12345";   // 예측 가능
```

**공격 방법**:
```bash
# Brute Force 공격
hashcat -m 16500 -a 0 jwt.txt wordlist.txt

# jwt_tool 사용
python3 jwt_tool.py <JWT> -C -d /path/to/wordlist.txt
```

**방어**:
```typescript
// ✅ 안전한 코드
import crypto from "crypto";
const secret = crypto.randomBytes(64).toString("hex");
// 또는 환경 변수에서 로드
const secret = process.env.JWT_SECRET; // 최소 32자 이상
```

---

### 4.3 Token Tampering (토큰 조작)

**취약점**: Payload 변조 후 서명 미검증

```json
// 원본 Payload
{
  "sub": "user123",
  "role": "user"
}

// 조작된 Payload
{
  "sub": "user123",
  "role": "admin"  // ← 권한 상승
}
```

**방어**:
```typescript
// ❌ 취약한 코드
const payload = jwt.decode(token); // 서명 검증 없음
if (payload.role === "admin") {
  // 관리자 권한 부여
}

// ✅ 안전한 코드
const payload = jwt.verify(token, secret); // 서명 검증
if (payload.role === "admin") {
  // 관리자 권한 부여
}
```

---

### 4.4 Expired Token (만료 시간 미검증)

**취약점**: 만료된 토큰 허용

```typescript
// ❌ 취약한 코드
const payload = jwt.decode(token);
// exp 체크 안 함

// ✅ 안전한 코드
const payload = jwt.verify(token, secret);
// verify()가 자동으로 exp 체크
```

---

### 4.5 JWT in URL (URL에 JWT 노출)

**취약점**: JWT를 URL 파라미터로 전송

```
❌ GET /api/data?token=eyJhbGci...
   - 브라우저 히스토리에 기록
   - 서버 로그에 기록
   - Referer 헤더로 누출
```

**방어**:
```
✅ GET /api/data
   Authorization: Bearer eyJhbGci...
```

---

## 🛡️ 5. 안전한 JWT 구현

### 5.1 TypeScript 예제 (Node.js)

```typescript
import jwt from "jsonwebtoken";

// 환경 변수에서 비밀키 로드
const SECRET = process.env.JWT_SECRET!; // 최소 32자
const ALGORITHM = "HS256";

// JWT 생성
function createToken(userId: string, role: string) {
  const payload = {
    sub: userId,
    role: role,
    iat: Math.floor(Date.now() / 1000),
    exp: Math.floor(Date.now() / 1000) + (60 * 60) // 1시간
  };

  return jwt.sign(payload, SECRET, { algorithm: ALGORITHM });
}

// JWT 검증
function verifyToken(token: string) {
  try {
    const payload = jwt.verify(token, SECRET, {
      algorithms: [ALGORITHM] // 알고리즘 명시
    });
    return payload;
  } catch (error) {
    if (error instanceof jwt.TokenExpiredError) {
      throw new Error("Token expired");
    }
    if (error instanceof jwt.JsonWebTokenError) {
      throw new Error("Invalid token");
    }
    throw error;
  }
}

// 미들웨어
function authMiddleware(req, res, next) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith("Bearer ")) {
    return res.status(401).json({ error: "No token provided" });
  }

  const token = authHeader.substring(7);

  try {
    const payload = verifyToken(token);
    req.user = payload; // 검증된 사용자 정보 저장
    next();
  } catch (error) {
    return res.status(401).json({ error: error.message });
  }
}
```

---

## 🔧 6. JWT 디버깅 도구

### 6.1 jwt.io
- **URL**: https://jwt.io
- **기능**: JWT 디코딩, 검증, 생성

### 6.2 jwt_tool (Python)
```bash
# 설치
git clone https://github.com/ticarpi/jwt_tool
cd jwt_tool

# 사용
python3 jwt_tool.py <JWT>
python3 jwt_tool.py <JWT> -C -d wordlist.txt  # Crack
python3 jwt_tool.py <JWT> -X a                # alg: none 공격
```

### 6.3 Burp Suite JWT 확장
- **JWT Editor**: JWT 편집 및 서명
- **JSON Web Tokens**: JWT 자동 탐지

---

## 📚 7. 참고 자료

- [JWT 공식 사이트](https://jwt.io)
- [RFC 7519 - JWT 표준](https://datatracker.ietf.org/doc/html/rfc7519)
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)

---

**Last Updated**: 2025-12-14
