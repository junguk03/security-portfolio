# 🛡️ OWASP Top 10 (2025)

> 가장 중요한 웹 애플리케이션 보안 위험 Top 10

---

## 📖 OWASP란?

**OWASP (Open Web Application Security Project)**는 웹 애플리케이션 보안을 개선하기 위한 비영리 재단입니다.

**OWASP Top 10**은 가장 위험한 웹 애플리케이션 보안 취약점을 3-4년마다 발표합니다.

> ⚠️ **2025년 업데이트**: OWASP Top 10:2025 (Final)이 공식 출시됐습니다. 이전 2021 버전은 **SUPERSEDED** 상태입니다. 2021 대비 주요 변경: Security Misconfiguration이 #5→#2로 상승, Software Supply Chain Failures(신규), Mishandling of Exceptional Conditions(신규), SSRF는 A01에 통합됐습니다.

---

## 🏆 OWASP Top 10 (2025)

| 순위 | 취약점 | 위험도 | 난이도 | 2021 대비 |
|------|--------|--------|--------|-----------|
| **A01** | Broken Access Control | 🔴 High | ⭐⭐⭐ | 유지 (SSRF 통합) |
| **A02** | Security Misconfiguration | 🟡 Medium | ⭐⭐ | ↑ #5→#2 |
| **A03** | Software Supply Chain Failures | 🔴 High | ⭐⭐⭐ | 🆕 신규 (A06 확장) |
| **A04** | Cryptographic Failures | 🔴 High | ⭐⭐⭐⭐ | ↓ #2→#4 |
| **A05** | Injection | 🔴 High | ⭐⭐⭐⭐ | ↓ #3→#5 |
| **A06** | Insecure Design | 🟡 Medium | ⭐⭐⭐ | ↓ #4→#6 |
| **A07** | Authentication Failures | 🔴 High | ⭐⭐⭐⭐ | 유지 (명칭 변경) |
| **A08** | Software or Data Integrity Failures | 🟡 Medium | ⭐⭐⭐⭐ | 유지 (명칭 변경) |
| **A09** | Security Logging & Alerting Failures | 🟢 Low | ⭐⭐ | 유지 (명칭 변경) |
| **A10** | Mishandling of Exceptional Conditions | 🟡 Medium | ⭐⭐⭐ | 🆕 신규 (SSRF 대체) |

---

## 1️⃣ A01:2025 Broken Access Control

### 설명
사용자가 권한 밖의 리소스에 접근할 수 있는 취약점. 2025년부터 SSRF(Server-Side Request Forgery)가 이 카테고리에 통합됐습니다.

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

## 2️⃣ A04:2025 Cryptographic Failures

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

## 3️⃣ A05:2025 Injection

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

## 4️⃣ A06:2025 Insecure Design

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

## 5️⃣ A02:2025 Security Misconfiguration ↑ (#5→#2)

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

## 6️⃣ A03:2025 Software Supply Chain Failures 🆕

### 설명
2025년 신규 카테고리. 취약점이 있는 라이브러리/프레임워크 사용(구 A06:2021)에서 확장돼, 전체 소프트웨어 의존성·빌드 시스템·배포 인프라 전반에 걸친 공급망 침해를 포함합니다. 관련 CWE: CWE-477, CWE-1104, CWE-1329, CWE-1395.

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
- 공급망 타이포스쿼팅(typosquatting) 패키지 설치
- 악성 패키지 업스트림 주입 (빌드 시스템 침해)

### 방어
```bash
# npm audit로 취약점 검사
npm audit

# 취약점 자동 수정
npm audit fix

# 의존성 업데이트
npm update

# lockfile 검증 및 SRI(Subresource Integrity) 활용
npm ci  # package-lock.json 기반으로 재현 가능한 빌드
```

---

## 7️⃣ A07:2025 Authentication Failures

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

## 8️⃣ A08:2025 Software or Data Integrity Failures

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

## 9️⃣ A09:2025 Security Logging & Alerting Failures

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

## 🔟 A10:2025 Mishandling of Exceptional Conditions 🆕

### 설명
2025년 신규 카테고리 (구 A10:2021 SSRF는 A01에 통합). 비정상적인 조건(예외·오류·논리 오류)이 발생했을 때 잘못된 처리로 인해 생기는 보안 취약점입니다. 관련 CWE: CWE-209(민감 정보를 포함한 에러 메시지), CWE-234(파라미터 누락 미처리), CWE-274(권한 부족 시 잘못된 처리), CWE-476(NULL 포인터 역참조), CWE-636(실패 시 열림 상태 — Fail Open).

### 예시
```typescript
// ❌ 취약한 코드: 예외 발생 시 권한 검증 우회
async function getResource(id: string, user: User) {
  try {
    return await db.getResource(id);
  } catch (e) {
    // 오류 시 null 반환 → 호출부에서 권한 없는 것으로 처리하지 않고 통과
    return null;
  }
}

// ❌ 취약한 코드: 오류 메시지에 스택 트레이스·내부 정보 노출
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message, stack: err.stack });
});
```

### 공격 시나리오
- 예외 발생 시 Fail-Open(권한 검증 건너뜀)으로 인한 인가 우회
- 에러 메시지에서 내부 경로·DB 구조 등 민감 정보 노출
- NULL 역참조로 인한 서비스 비정상 종료

### 방어
```typescript
// ✅ 안전한 코드: 예외 시 명시적으로 거부(Fail Secure)
async function getResource(id: string, user: User) {
  try {
    const resource = await db.getResource(id);
    if (!resource || resource.ownerId !== user.id) {
      throw new ForbiddenError();
    }
    return resource;
  } catch (e) {
    if (e instanceof ForbiddenError) throw e;
    throw new InternalError(); // 민감 정보 미노출
  }
}

// ✅ 안전한 코드: 프로덕션에서 일반 메시지만 반환
app.use((err, req, res, next) => {
  logger.error(err); // 내부 로그에만 상세 기록
  const status = err.status ?? 500;
  res.status(status).json({ error: status < 500 ? err.message : 'Internal Server Error' });
});
```

> ⚠️ **SSRF (구 A10:2021)**: SSRF는 A01:2025 Broken Access Control에 통합됐습니다. 내부 URL 접근 차단·도메인 화이트리스트 등 SSRF 방어는 A01 섹션을 참고하세요.

---

## 📚 참고 자료

- [OWASP Top 10:2025 (공식)](https://owasp.org/Top10/2025/)
- [OWASP Top 10:2025 GitHub](https://github.com/OWASP/Top10/tree/master/2025)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

---

**Last Updated**: 2026-05-12
**OWASP Top 10 Version**: 2025 (2021은 SUPERSEDED)
