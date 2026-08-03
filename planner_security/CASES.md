# Planner 보안 사례

플래너 프로젝트에서 실제로 발견된 보안 이슈와 해결 기록

---

## 사례 1 — IDOR (Insecure Direct Object Reference)

- **분류**: OWASP A01 Broken Access Control / CWE-639
- **위치**: `page.tsx:198, 234, 242`
- **문제**: `eq('id', eventId)`만 사용 — 다른 유저의 이벤트도 ID만 알면 삭제/수정 가능
- **공격 시나리오**: 인증된 공격자가 이벤트 ID 추측 또는 다른 경로로 획득 후 다른 유저 데이터 조작

**취약 코드**
```ts
await supabase.from('events').delete().eq('id', eventId);
await supabase.from('events').update({ done }).eq('id', eventId);
```

**수정 코드**
```ts
await supabase.from('events')
  .delete()
  .eq('id', eventId)
  .eq('user_id', user.id);
```

**추가 조치**
- Supabase 대시보드에서 `events` 테이블 RLS 활성화
- 정책: `auth.uid() = user_id`

---

## 사례 2 — 보안 헤더 누락

- **분류**: OWASP A05 Security Misconfiguration / CWE-693
- **위치**: `next.config.ts`
- **문제**: CSP, HSTS, X-Frame-Options 등 보안 헤더 전무

**해결**: 모든 페이지에 헤더 7개 적용

| 헤더 | 역할 |
|------|------|
| `X-Content-Type-Options: nosniff` | MIME 스니핑 차단 |
| `X-Frame-Options: DENY` | 클릭재킹 차단 |
| `X-XSS-Protection` | 구형 브라우저 XSS 필터 |
| `Referrer-Policy` | referrer 최소화 |
| `Permissions-Policy` | 카메라/마이크/위치 API 차단 |
| `Strict-Transport-Security` | HTTPS 강제 (2년) |
| `Content-Security-Policy` | 허용 출처(Supabase, GA, Google Fonts)만 |

**검증**: [securityheaders.com](https://securityheaders.com) A 등급 목표

---

## 사례 3 — 패키지 취약점 7개

- **분류**: OWASP A06 Vulnerable and Outdated Components / CWE-1104
- **도구**: `npm audit`

| 패키지 | 심각도 | 내용 |
|--------|--------|------|
| `next` | HIGH | HTTP request smuggling, Server Components DoS, image disk cache 무한 성장 |
| `flatted` | HIGH | Prototype Pollution + 재귀 DoS |
| `minimatch` | HIGH | ReDoS |
| `picomatch` | HIGH | ReDoS + Method Injection |
| `postcss` | moderate | `</style>` 미이스케이프 → XSS |
| `ajv` | moderate | ReDoS |
| `brace-expansion` | moderate | 메모리 고갈 DoS |

**해결**
- Next.js 15 → 16 업그레이드
- `package.json` overrides로 postcss 8.5.10+ 강제 적용
- 나머지는 `npm audit fix` 자동 처리
- 결과: 7개 전부 제거

---

## 사례 4 — Supabase Key 혼동 (참고)

- **분류**: OWASP A02 Cryptographic Failures / CWE-798
- **개념**: Supabase는 두 종류 키 제공
  - `anon` (public) — 클라이언트 사용 OK, RLS로 권한 제어
  - `service_role` (secret) — 서버 전용, **절대** 클라이언트 노출 금지

**올바른 사용**
```bash
# .env
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...     # 클라이언트 OK
SUPABASE_SERVICE_ROLE_KEY=eyJ...          # 서버 전용 (NEXT_PUBLIC_ 없음)
```

**금지 패턴**
```bash
# ❌ service_role을 NEXT_PUBLIC_ 접두사로 — 클라이언트 번들에 박힘
NEXT_PUBLIC_SUPABASE_SERVICE_ROLE=eyJ...
```

---

## 진행 상황

| Step | 내용 | 상태 |
|------|------|------|
| 1 | npm 취약점 7개 제거 | ✅ |
| 2 | 보안 헤더 7개 추가 | ✅ |
| 3 | `page.tsx`에 user_id 필터 추가 | ⏳ |
| 4 | Supabase RLS 활성화 확인 | ⏳ |

---

Last Updated: 2026-05-11
