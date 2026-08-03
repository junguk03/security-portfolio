# Planner 보안 취약점 분석

Vercel 배포 중인 플래너 앱 대상 보안 감사 결과

---

## 취약점 요약

| 심각도 | 항목 | 상태 |
|--------|------|------|
| 🔴 HIGH | 패키지 취약점 7개 (npm audit) | ✅ 해결 |
| 🟡 MEDIUM | DB 쿼리 user_id 필터 누락 | ⏳ 미해결 |
| 🟡 MEDIUM | 보안 헤더 미설정 (next.config.ts) | ✅ 해결 |

---

## 🔴 HIGH — 패키지 취약점

`npm audit` 결과: **7개** (high 4, moderate 3)

| 패키지 | 심각도 | 내용 |
|--------|--------|------|
| `next` | HIGH | HTTP request smuggling, Server Components DoS, image disk cache 무한 성장 |
| `flatted` | HIGH | Prototype Pollution + 재귀 DoS |
| `minimatch` | HIGH | ReDoS (정규식 폭발) |
| `picomatch` | HIGH | ReDoS + Method Injection |
| `postcss` | moderate | `</style>` 미이스케이프 → XSS |
| `ajv` | moderate | ReDoS |
| `brace-expansion` | moderate | 메모리 고갈 DoS |

**Next.js HTTP request smuggling이 가장 심각** — Vercel 배포 환경에서 rewrites를 통해 공격자가 다른 유저의 요청을 가로챌 수 있음.

### Step 1 — 패키지 업데이트 ✅ 완료 (2026-04-30)

- Next.js 15 → 16 업그레이드 (HTTP request smuggling 등 HIGH 패치)
- `package.json` overrides로 postcss 8.5.10+ 강제 적용
- 나머지 패키지는 `npm audit fix`로 자동 업데이트
- 결과: 취약점 7개 (high 4, moderate 3) 전부 제거

---

## 🟡 MEDIUM — DB 쿼리 user_id 필터 누락

**위치:** `page.tsx:198`, `page.tsx:234`, `page.tsx:242`

다른 유저의 이벤트를 id만 알면 삭제/수정할 수 있는 IDOR(Insecure Direct Object Reference) 취약점.

```ts
// 현재 (취약)
await supabase.from('events').delete().eq('id', id);
await supabase.from('events').update({ done: ... }).eq('id', eventId);

// 수정 후 (안전)
await supabase.from('events').delete().eq('id', id).eq('user_id', user.id);
await supabase.from('events').update({ done: ... }).eq('id', eventId).eq('user_id', user.id);
```

### Step 3 — user_id 필터 추가

- [ ] `page.tsx:198` 수정
- [ ] `page.tsx:234` 수정
- [ ] `page.tsx:242` 수정

### Step 4 — Supabase RLS 활성화 확인

- [ ] Supabase 대시보드 → Table Editor → RLS 활성화 여부 확인
- [ ] `events` 테이블에 `user_id = auth.uid()` 정책 설정 확인

---

## 🟡 MEDIUM — 보안 헤더 미설정

**위치:** `next.config.ts`

CSP, X-Frame-Options, HSTS 등 보안 헤더 없음.

### Step 2 — next.config.ts 보안 헤더 추가 ✅ 완료 (2026-05-01)

모든 페이지에 헤더 7개 적용:

| 헤더 | 역할 |
|------|------|
| `X-Content-Type-Options` | MIME 타입 스니핑 차단 |
| `X-Frame-Options: DENY` | iframe 삽입(클릭재킹) 차단 |
| `X-XSS-Protection` | 구형 브라우저 XSS 필터 활성화 |
| `Referrer-Policy` | 외부 요청 시 referrer 정보 최소화 |
| `Permissions-Policy` | 카메라/마이크/위치 API 차단 |
| `Strict-Transport-Security` | HTTPS 강제 (2년) |
| `Content-Security-Policy` | 허용된 출처(Supabase, GA, Google Fonts)만 허용 |

---

## 해결 순서

1. **Step 1** — `npm audit fix` + `next@latest` 업데이트
2. **Step 2** — `next.config.ts` 보안 헤더 추가
3. **Step 3** — DB 쿼리 `user_id` 필터 추가
4. **Step 4** — Supabase RLS 활성화 확인

---

Last Updated: 2026-04-30
