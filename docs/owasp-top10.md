# 🛡️ OWASP Top 10 (2021)

> 가장 중요한 웹 애플리케이션 보안 위험 Top 10

---

## 📖 OWASP란?

**OWASP (Open Web Application Security Project)**는 웹 애플리케이션 보안을 개선하기 위한 비영리 재단입니다.

**OWASP Top 10**은 가장 위험한 웹 애플리케이션 보안 취약점을 3-4년마다 발표합니다.

---

## 🏆 OWASP Top 10 (2021)

| 순위 | 취약점 | 위험도 | 난이도 |
|------|--------|--------|--------|
| **A01** | Broken Access Control | 🔴 High | ⭐⭐⭐ |
| **A02** | Cryptographic Failures | 🔴 High | ⭐⭐⭐⭐ |
| **A03** | Injection | 🔴 High | ⭐⭐⭐⭐ |
| **A04** | Insecure Design | 🟡 Medium | ⭐⭐⭐ |
| **A05** | Security Misconfiguration | 🟡 Medium | ⭐⭐ |
| **A06** | Vulnerable Components | 🟡 Medium | ⭐⭐⭐ |
| **A07** | Authentication Failures | 🔴 High | ⭐⭐⭐⭐ |
| **A08** | Software & Data Integrity | 🟡 Medium | ⭐⭐⭐⭐ |
| **A09** | Logging & Monitoring | 🟢 Low | ⭐⭐ |
| **A10** | SSRF | 🔴 High | ⭐⭐⭐⭐ |

---

## 1️⃣ A01: Broken Access Control

### 설명
사용자가 권한 밖의 리소스에 접근할 수 있는 취약점

### 예시
```typescript
// ❌ 취약한 코드
app.get('/api/user/:id', (req, res) => {
  const user = getUserById(req.params.id);
  res.json(user); // 누구나 다른 사용자 정보 조회 가능
});
```

### 공격 시나리오
- **IDOR (Insecure Direct Object Reference)**:
  ```
  GET /api/user/123  ← 내 ID
  GET /api/user/124  ← 다른 사용자 ID (접근 가능!)
  ```

- **Path Traversal**:
  ```
  GET /api/file?path=../../../../etc/passwd
  ```

### 방어
```typescript
// ✅ 안전한 코드
app.get('/api/user/:id', authMiddleware, (req, res) => {
  if (req.user.id !== req.params.id && req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Forbidden' });
  }
  const user = getUserById(req.params.id);
  res.json(user);
});
```

---

## 2️⃣ A02: Cryptographic Failures

### 설명
민감 데이터를 암호화하지 않거나 약한 암호화 사용

### 예시
```typescript
// ❌ 취약한 코드
app.post('/login', (req, res) => {
  const user = db.query(`SELECT * FROM users WHERE password = '${req.body.password}'`);
  // 평문 비밀번호 저장!
});
```

### 공격 시나리오
- 평문 비밀번호 저장
- HTTP로 민감 데이터 전송 (HTTPS 미사용)
- 약한 해시 함수 사용 (MD5, SHA1)

### 방어
```typescript
// ✅ 안전한 코드
import bcrypt from 'bcrypt';

app.post('/register', async (req, res) => {
  const hashedPassword = await bcrypt.hash(req.body.password, 10);
  db.query(`INSERT INTO users (password) VALUES (?)`, [hashedPassword]);
});

app.post('/login', async (req, res) => {
  const user = db.query(`SELECT * FROM users WHERE username = ?`, [req.body.username]);
  const isValid = await bcrypt.compare(req.body.password, user.password);
});
```

---

## 3️⃣ A03: Injection

### 설명
신뢰할 수 없는 데이터를 명령어나 쿼리에 포함시켜 실행

### 예시: SQL Injection
```typescript
// ❌ 취약한 코드
app.get('/user', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ${req.query.id}`;
  db.query(query); // SQL Injection 취약점!
});
```

### 공격 시나리오
```
GET /user?id=1 OR 1=1  ← 모든 사용자 조회
GET /user?id=1; DROP TABLE users--  ← 테이블 삭제
```

### 방어
```typescript
// ✅ 안전한 코드 (Prepared Statement)
app.get('/user', (req, res) => {
  const query = `SELECT * FROM users WHERE id = ?`;
  db.query(query, [req.query.id]); // 파라미터 바인딩
});
```

### 기타 Injection 공격
- **Command Injection**: `exec('ping ' + userInput)`
- **XSS (Cross-Site Scripting)**: `<script>alert('XSS')</script>`
- **LDAP Injection**
- **NoSQL Injection**

---

## 4️⃣ A04: Insecure Design

### 설명
설계 단계에서부터 보안이 고려되지 않은 경우

### 예시
- 비밀번호 찾기 시 보안 질문만 사용
- Rate Limiting 없음
- 다중 인증(MFA) 미지원

### 방어
- 위협 모델링 수행
- 보안 설계 패턴 적용
- "Secure by Design" 원칙

---

## 5️⃣ A05: Security Misconfiguration

### 설명
잘못된 보안 설정으로 인한 취약점

### 예시
```typescript
// ❌ 취약한 코드
app.use((err, req, res, next) => {
  res.status(500).json({
    error: err.message,
    stack: err.stack  // 스택 트레이스 노출!
  });
});
```

### 공격 시나리오
- 기본 계정/비밀번호 사용
- 불필요한 기능 활성화 (디버그 모드)
- 에러 메시지에 민감 정보 노출
- CORS 설정 오류

### 방어
```typescript
// ✅ 안전한 코드
if (process.env.NODE_ENV === 'production') {
  app.use((err, req, res, next) => {
    res.status(500).json({ error: 'Internal Server Error' });
  });
} else {
  // 개발 환경에서만 상세 에러
  app.use((err, req, res, next) => {
    res.status(500).json({ error: err.message, stack: err.stack });
  });
}
```

---

## 6️⃣ A06: Vulnerable and Outdated Components

### 설명
취약점이 있는 라이브러리/프레임워크 사용

### 예시
```json
// package.json
{
  "dependencies": {
    "express": "3.0.0",  // 오래된 버전!
    "lodash": "4.17.0"   // 취약점 있음!
  }
}
```

### 공격 시나리오
- 알려진 CVE(Common Vulnerabilities and Exposures) 악용
- Prototype Pollution (lodash 취약점)

### 방어
```bash
# npm audit로 취약점 검사
npm audit

# 취약점 자동 수정
npm audit fix

# 의존성 업데이트
npm update
```

---

## 7️⃣ A07: Identification and Authentication Failures

### 설명
인증 및 세션 관리 취약점

### 예시
```typescript
// ❌ 취약한 코드
app.post('/login', (req, res) => {
  const user = db.query(`SELECT * FROM users WHERE username = ?`, [req.body.username]);
  if (user && user.password === req.body.password) {
    // 세션 ID 예측 가능!
    res.cookie('sessionId', user.id);
  }
});
```

### 공격 시나리오
- Brute Force 공격 (Rate Limiting 없음)
- 세션 ID 예측
- 약한 비밀번호 정책

### 방어
```typescript
// ✅ 안전한 코드
import session from 'express-session';
import rateLimit from 'express-rate-limit';

// Rate Limiting
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15분
  max: 5 // 최대 5번 시도
});

app.post('/login', loginLimiter, async (req, res) => {
  const user = db.query(`SELECT * FROM users WHERE username = ?`, [req.body.username]);
  const isValid = await bcrypt.compare(req.body.password, user.password);

  if (isValid) {
    req.session.userId = user.id; // 안전한 세션 관리
  }
});
```

---

## 8️⃣ A08: Software and Data Integrity Failures

### 설명
무결성 검증 없이 신뢰할 수 없는 소스에서 코드/데이터 사용

### 예시
```html
<!-- ❌ 취약한 코드 -->
<script src="https://cdn.example.com/library.js"></script>
<!-- 무결성 검증 없음! -->
```

### 공격 시나리오
- CDN 해킹 시 악성 코드 주입
- Deserialization 공격

### 방어
```html
<!-- ✅ 안전한 코드 (SRI 사용) -->
<script
  src="https://cdn.example.com/library.js"
  integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
  crossorigin="anonymous">
</script>
```

---

## 9️⃣ A09: Security Logging and Monitoring Failures

### 설명
보안 로그 및 모니터링 부족

### 예시
```typescript
// ❌ 취약한 코드
app.post('/login', (req, res) => {
  // 로그인 실패 로그 없음!
  if (!isValidUser(req.body)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

### 방어
```typescript
// ✅ 안전한 코드
import winston from 'winston';

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [new winston.transports.File({ filename: 'security.log' })]
});

app.post('/login', (req, res) => {
  if (!isValidUser(req.body)) {
    logger.warn('Failed login attempt', {
      username: req.body.username,
      ip: req.ip,
      timestamp: new Date()
    });
    return res.status(401).json({ error: 'Invalid credentials' });
  }
});
```

---

## 🔟 A10: Server-Side Request Forgery (SSRF)

### 설명
서버가 공격자가 지정한 URL로 요청을 보내는 취약점

### 예시
```typescript
// ❌ 취약한 코드
app.get('/fetch', async (req, res) => {
  const response = await fetch(req.query.url); // SSRF 취약점!
  res.send(await response.text());
});
```

### 공격 시나리오
```
GET /fetch?url=http://localhost:6379  ← Redis 접근
GET /fetch?url=http://169.254.169.254/latest/meta-data/  ← AWS 메타데이터
```

### 방어
```typescript
// ✅ 안전한 코드
const allowedDomains = ['api.example.com', 'cdn.example.com'];

app.get('/fetch', async (req, res) => {
  const url = new URL(req.query.url);

  // 도메인 화이트리스트 검증
  if (!allowedDomains.includes(url.hostname)) {
    return res.status(400).json({ error: 'Invalid domain' });
  }

  // 내부 IP 차단
  if (url.hostname === 'localhost' || url.hostname.startsWith('127.') || url.hostname.startsWith('192.168.')) {
    return res.status(400).json({ error: 'Internal IP blocked' });
  }

  const response = await fetch(url);
  res.send(await response.text());
});
```

---

## 📚 참고 자료

- [OWASP Top 10 2021](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

---

**Last Updated**: 2025-12-14
**OWASP Top 10 Version**: 2021
