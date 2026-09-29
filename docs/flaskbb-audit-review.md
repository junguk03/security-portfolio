# FlaskBB 37개 발견 사항 재검토 — 제보 가능성 및 공격 가능 여부

**검토 일자**: 2026-09-29
**목적**: 1차/2차 감사에서 발견된 항목들의 실제 공격 가능 여부를 소스코드 대조하여 재검증하고, FlaskBB 메인테이너에게 제보 가능한 항목을 선별한다.

---

## 분류 기준

| 등급 | 의미 | 제보 여부 |
|------|------|-----------|
| **A — 실제 공격 가능** | 기본 설정에서 공격 시연 가능, PoC 작성 가능 | ✅ 제보 가능 |
| **B — 조건부 공격 가능** | 특정 설정/환경에서만 트리거되지만 실제 위협 | ✅ 제보 가능 (조건 명시) |
| **C — 보안 강화 권장** | 즉시 공격 불가, Best Practice 미준수 | ⚠️ 강화 요청으로 제보 가능 |
| **D — 제보 불가** | 오탐, 설계 의도, 중복, 또는 현재 코드에서 안전 | ❌ 제보 불가 |

---

## 중복 항목 정리

소스코드 대조 결과, 1차와 2차 감사 사이에 **6쌍의 중복**이 확인되었다:

| 1차 | 2차 | 내용 | 통합 |
|-----|-----|------|------|
| M-04 | M-20 | 패스워드 리셋 토큰 재사용 | → M-04로 통합 |
| M-08 | L-13 | 보안 헤더 부재 | → M-08로 통합 |
| M-10 | M-21 | 로그인 시도 카운터 레이스 컨디션 | → M-10으로 통합 |
| L-01 | M-17 | Remember Cookie HttpOnly=False | → L-01로 통합 |
| M-12 | M-18 | 이메일 열거 | → M-12로 통합 |
| M-13 | I-10 | SimpleCache 권한 캐시 불일치 | → M-13으로 통합 |

**중복 제거 후 고유 발견: 34개**

---

## A등급 — 실제 공격 가능 (제보 ✅)

### 1. M-03. GET 로그아웃 CSRF ← 즉시 공격 가능

- **위치**: `auth/views.py:77-83`
- **검증**: `Logout` 클래스의 `def get(self)` 메서드가 `logout_user()` 호출. CSRF 토큰 검증 없음.
- **PoC**: `<img src="/auth/logout">` 를 게시물에 삽입 → 조회한 모든 사용자 자동 로그아웃
- **공격 가능**: ✅ 기본 설정에서 즉시 가능
- **영향**: 서비스 방해 (모든 사용자 강제 로그아웃)
- **제보 가치**: ⭐⭐⭐ (CWE-352, 표준적인 CSRF 취약점)

### 2. M-04/M-20. 패스워드 리셋 토큰 재사용 ← 실제 공격 가능

- **위치**: `auth/services/password.py:55-66`
- **검증**: `reset_password()` 메서드에서 토큰 사용 후 무효화 로직이 없음. JWT의 `exp` 클레임만 검증.
- **PoC**: 1) 리셋 이메일 수신 → 비밀번호 변경 → 2) 동일 토큰으로 1시간 내 재변경 가능
- **공격 가능**: ✅ 이메일을 가로챈 공격자가 피해자의 비밀번호를 반복 변경
- **영향**: 계정 탈취
- **제보 가치**: ⭐⭐⭐ (CWE-613)

### 3. M-05. BlockTooManyFailedLogins 미등록 ← 실제 공격 가능

- **위치**: `auth/plugins.py` 전체 검토
- **검증 결과**:
  - `MarkFailedLogin` → `flaskbb_authentication_failed` 훅에 등록됨 (line 60-62) ✅
  - `ClearFailedLogins` → `flaskbb_post_authenticate` 훅에 등록됨 (line 51) ✅
  - `BlockTooManyFailedLogins` → **어떤 훅에도 등록되지 않음** ❌
  - 클래스가 정의만 되어 있고 (`authentication.py:53-77`), `plugins.py`의 import 목록에도 없음
- **결과**: `login_attempts`는 증가하지만 검사하는 코드가 실행되지 않아, 계정 잠금이 작동하지 않음
- **완화**: Flask-Limiter가 IP 기반 rate limit 적용 중 (`auth/views.py:447`)
- **공격 가능**: ✅ 다중 IP (프록시/봇넷)에서 무제한 brute force 가능
- **제보 가치**: ⭐⭐⭐⭐ (CWE-307, 인증 보호 핵심 결함)

### 4. M-12/M-18. 이메일 열거 ← 즉시 공격 가능

- **위치**: `auth/views.py:218-234`
- **검증**:
  - 유효 이메일: `flash("Email sent!", "info")` + `redirect` (line 233-234)
  - 무효 이메일: `flash("You have entered an username or email...", "danger")` (line 225-230)
  - 응답 코드도 다름: 성공 시 302, 실패 시 200 (폼 재렌더링)
- **PoC**: 이메일 목록에 대해 `/auth/reset-password` POST → 응답 차이로 가입 여부 확인
- **공격 가능**: ✅ 자동화 스크립트로 대량 이메일 확인 가능
- **제보 가치**: ⭐⭐⭐ (CWE-204)

### 5. M-14. 비인증 마크다운 프리뷰 DoS ← 즉시 공격 가능

- **위치**: `forum/views.py:1125-1132`
- **검증**:
  - `MarkdownPreview` 클래스에 `login_required` 데코레이터 없음
  - `force_login_if_needed()` (line 1286)는 `current_forum` 컨텍스트 필요 → `/markdown` 경로에서는 `current_forum = None` → 체크 우회
  - `MAX_CONTENT_LENGTH = 32 * 1024 * 1024` (32MB) → 비인증 사용자가 32MB 마크다운 렌더링 요청 가능
- **PoC**: `curl -X POST -d "text=$(python3 -c 'print("**"*1000000)')" http://target/markdown`
- **공격 가능**: ✅ 비인증 사용자가 반복적으로 대용량 마크다운 렌더링 → CPU/메모리 소모
- **제보 가치**: ⭐⭐⭐ (CWE-400)

### 6. M-16. 첨부파일 접근 제어 부재 ← 즉시 공격 가능

- **위치**: `upload/views.py:34-50`
- **검증**: 코드 주석에 `"Attachments are not yet access control checked"` (line 37) 명시. 인증/권한 검사 없이 `send_from_directory()` 직접 호출.
- **PoC**: 비공개 포럼의 첨부파일 URL을 알면 (`/uploads/attachments/{filename}/{display_name}`), 비로그인 상태에서도 다운로드 가능
- **공격 가능**: ✅ URL 패턴 추측 또는 로그 분석으로 비공개 첨부파일 접근
- **제보 가치**: ⭐⭐⭐⭐ (CWE-862, 개발자가 이미 인지한 TODO이므로 우선순위 상향 가능)

### 7. M-19. PostgreSQL 사용자명 대소문자 우회 ← 공격 가능

- **위치**: `auth/services/registration.py:108-112`
- **검증**:
  ```python
  sa.func.lower(self.users.username) == user_info.username
  ```
  `lower()`가 DB 컬럼에만 적용, 입력값에는 미적용. PostgreSQL의 `=`는 대소문자 구분.
  - `LOWER('admin') = 'Admin'` → PostgreSQL에서 `FALSE`
  - 동일 파일의 `EmailUniquenessValidator` (line 133)도 같은 패턴: `sa.func.lower(self.users.email) == user_info.email`
- **PoC**: "admin" 존재 시 "Admin" 또는 "ADMIN"으로 가입 → 유일성 검증 통과 → 사칭 계정 생성
- **공격 가능**: ✅ (PostgreSQL 사용 시)
- **참고**: SQLite (기본 DB)에서는 대소문자 무시 collation으로 안전
- **추가 발견**: EmailUniquenessValidator도 동일 버그 → 같은 이메일의 대소문자 변형으로 중복 가입 가능
- **제보 가치**: ⭐⭐⭐⭐ (CWE-178, 명확한 코드 버그)

### 8. M-01. LIKE 와일드카드 인젝션 (검색) ← 공격 가능

- **위치**: `search/service.py:74`
- **검증**: `model.username.ilike(f"%{author}%")` — `escape_like()` 미사용. 동일 코드베이스 `user/views.py:280`에서는 올바르게 사용 중.
- **PoC**: 검색 작성자에 `%` 입력 → 모든 사용자명 매칭. `___` → 3글자 사용자명만 열거.
- **공격 가능**: ✅ (정보 유출 수준)
- **제보 가치**: ⭐⭐ (CWE-943, 영향 제한적이나 수정 간단)

### 9. L-14. Host Header Poisoning ← 공격 가능

- **위치**: `configs/default.py:63`, `app.py:233-246`
- **검증**:
  - `TRUSTED_HOSTS = None` + `SERVER_NAME`도 미설정 (기본)
  - `app.py`에서 경고 로그만 출력, 요청 차단하지 않음
  - `url_for(_external=True)`가 공격자의 `Host` 헤더 반영
- **PoC**: `Host: evil.com` 헤더로 패스워드 리셋 요청 → 이메일의 리셋 링크가 `http://evil.com/...`으로 생성 → 피해자 클릭 시 토큰 유출
- **공격 가능**: ✅ 기본 설정에서 가능
- **제보 가치**: ⭐⭐⭐ (CWE-644)

---

## B등급 — 조건부 공격 가능 (제보 ✅, 조건 명시)

### 10. C-01. 하드코딩된 SECRET_KEY ← 미변경 배포 시 공격 가능

- **위치**: `configs/default.py:178, 184`
- **검증**: `SECRET_KEY = "secret key"`, `WTF_CSRF_SECRET_KEY = "reallyhardtoguess"`
- **완화 요소**: `flaskbb install` 명령이 랜덤 키를 생성하여 별도 설정 파일에 기록
- **조건**: 설치 프로세스를 거치지 않고 기본 설정으로 배포한 경우
- **공격 가능**: ✅ (조건 충족 시) — 세션 위조, JWT 생성, CSRF 우회 모두 가능
- **제보 가치**: ⭐⭐⭐ (시작 시 기본값 감지 → 거부하는 안전장치 요청)

### 11. H-01. SVG 첨부파일 XSS ← 관리자 설정 변경 시 공격 가능

- **위치**: `upload/views.py:47-48`
- **검증**:
  - `ATTACHMENT_TYPES` 기본값: `"png, jpg, jpeg, gif, pdf"` — SVG 미포함 ✅
  - `as_attachment=not attachment.is_image` → SVG(`image/svg+xml`)는 `is_image=True` → 인라인 서빙
  - SVG 내 `<script>` 태그가 브라우저에서 실행됨
- **조건**: 관리자가 `ATTACHMENT_TYPES`에 `svg` 추가 시
- **공격 가능**: ✅ (조건 충족 시) — 저장형 XSS
- **제보 가치**: ⭐⭐⭐ (방어적 코딩: `image/svg+xml`을 인라인 서빙에서 제외하는 화이트리스트 요청)

### 12. M-13/I-10. SimpleCache 권한 캐시 불일치 ← 멀티워커 배포 시

- **위치**: `user/models.py:381-401`, `configs/default.py:241`
- **검증**: `CACHE_TYPE = "SimpleCache"` (프로세스 내 메모리), `CACHE_DEFAULT_TIMEOUT = 60`
- **조건**: Gunicorn 등 멀티워커 배포에서 `SimpleCache` 사용 시
- **공격 시나리오**: 관리자가 사용자를 밴 → 해당 사용자가 다른 워커에 요청 → 최대 60초간 기존 권한으로 접근
- **공격 가능**: ✅ (조건 충족 시)
- **제보 가치**: ⭐⭐ (문서에 프로덕션에서 Redis 캐시 필수 사용 권장 요청)

### 13. M-15. 아바타 PIL 검증 우회 ← 기능 버그로 보안 검증 무력화

- **위치**: `utils/uploads.py:185, 211-214`
- **검증**:
  - Line 185: `str.lower(parser.image.format or "")` → PIL은 JPEG를 `"JPEG"`로 반환 → `"jpeg"`
  - Line 212-214: `img_info["content_type"] not in config["AVATAR_EXTENSIONS"]`
  - `AVATAR_EXTENSIONS = ["jpg", "png", "gif"]` → `"jpeg" not in ["jpg", "png", "gif"]` = **True**
  - 결과: 정상 JPEG 파일이 PIL 콘텐츠 검증에서 거부됨
- **영향**: PIL 기반 이미지 콘텐츠 타입 검증이 JPEG에 대해 무력화. 확장자 기반 검증만 남음.
- **공격 가능**: △ (직접적인 공격보다는 방어 계층 약화)
- **제보 가치**: ⭐⭐⭐ (명확한 코드 버그, 수정 간단: `"jpeg"` → `["jpg", "jpeg"]` 변환 매핑 추가)

---

## C등급 — 보안 강화 권장 (제보 ⚠️)

### 14. M-02. LIKE 와일드카드 인젝션 (관리자)

- **검증**: `management/forms.py:114` — 관리자 전용 페이지이므로 공격 표면 제한
- **공격 가능**: △ (관리자 계정 탈취 전제)
- **제보**: M-01과 합쳐서 보고 가능

### 15. M-08/L-13. 보안 헤더 전면 부재

- **검증**: CSP, X-Frame-Options, X-Content-Type-Options, HSTS 등 미설정
- **공격 가능**: 단독으로는 불가, 다른 취약점과 결합 시 방어 약화
- **제보**: ⚠️ 강화 요청 (모든 보안 스캐너가 지적하는 항목)

### 16. M-09. 비원자적 카운터 (Post/Topic/Forum)

- **검증**: `forum/models.py:423-425` — `user.post_count += 1` 등
- **공격 가능**: △ (카운트 조작만 가능, 보안 영향 미미)
- **제보**: ⚠️ 데이터 무결성 버그로 보고 가능

### 17. M-10/M-21. 로그인 시도 카운터 레이스 컨디션

- **검증**: `authentication.py:119` — `user.login_attempts += 1`
- **핵심**: M-05에서 확인한 대로 `BlockTooManyFailedLogins`가 미등록이므로, 이 카운터는 **검사되지 않는 값**을 증가시키고 있음. 레이스 컨디션이 있지만 보안 영향 없음 (잠금 자체가 작동하지 않으므로).
- **공격 가능**: ❌ (M-05가 수정되기 전까지 의미 없음)
- **제보**: M-05에 포함하여 보고

### 18. L-01/M-17. Remember Cookie HttpOnly=False

- **검증**: `configs/default.py:220` — 기본값 `False`. Docker 설정에서는 `True`.
- **공격 가능**: XSS 존재 시에만 → 단독 공격 불가
- **제보**: ⚠️ 기본값 변경 요청

### 19. L-02. 패스워드 복잡도 미검증

- **검증**: `auth/forms.py` — `InputRequired()` + `EqualTo()` 만 존재. 길이/복잡도 요구 없음.
- **공격 가능**: 직접적 공격 아님, weak password 허용
- **제보**: ⚠️ 최소 길이 요구사항 추가 요청

### 20. L-03. PREFERRED_URL_SCHEME=http

- **검증**: `configs/default.py:45` — 개발용 기본값으로 합리적. 프로덕션에서 HTTPS 설정 필요.
- **제보**: ⚠️ 문서 강화

### 21. L-05. 예측 가능한 아바타 파일명

- **검증**: `utils/uploads.py:35` — `avatar_{username}.{ext}`. 아바타는 공개 콘텐츠.
- **공격 가능**: ❌ (아바타 URL은 이미 프로필에 공개)
- **제보**: △ 우선순위 낮음

### 22. L-11. 세션 쿠키 보안 플래그 미설정

- **검증**: Flask 기본값 `SESSION_COOKIE_HTTPONLY=True` (OK), `SECURE=False`, `SAMESITE=None`
- **제보**: ⚠️ `SESSION_COOKIE_SAMESITE="Lax"` 기본값 추가 요청

### 23. L-12. update_lastseen 매 요청 DB 커밋

- **검증**: `app.py:414-420` — 보안보다 성능 이슈
- **제보**: △ 성능 개선 제안으로만

---

## D등급 — 제보 불가 (오탐/설계 의도/이론적)

### 24. M-06. Rate Limiting 토글 미작동 ← 오탐

- **검증 결과**: `auth/views.py:427-433`에서 `limiter.check()` 주석 처리되어 있으나,
  **line 447에서 `limiter.limit(login_rate_limit, ...)(auth)` 가 블루프린트 전체에 rate limiting을 적용 중**.
  Rate limiting은 실제로 **작동하고 있음**. 주석 처리된 코드는 DB 설정 기반 토글 기능이 미완성인 것이지, rate limiting 자체가 미작동하는 것이 아님.
- **결론**: 관리자 패널 토글이 안 먹는 기능 버그이지, 보안 취약점이 아님. Rate limiting은 항상 활성화됨 (오히려 더 안전한 상태).
- **제보 불가**: 기능 버그로 보고할 수 있으나 보안 취약점이 아님.

### 25. M-07. 토큰 만료 시간 계산 시점 ← 현재 코드에서 안전

- **검증**: `tokens/serializer.py:47` — `__init__` 시점에 `now() + timedelta` 계산
- **팩토리 패턴 확인**: `auth/services/factories.py`에서 요청마다 새 인스턴스 생성
- **결론**: 현재 코드에서 캐싱되지 않아 안전. 코드 냄새(code smell)이지 취약점이 아님.
- **제보 불가**: 이론적 위험에 불과.

### 26. M-11. 플러그인 훅 Markup 인젝션 ← 설계 의도

- **검증**: `plugins/utils.py:127` — `return Markup(result)`
- **결론**: 플러그인은 서버 사이드 코드로 전체 시스템 접근 권한을 가짐. 플러그인이 HTML을 렌더링하는 것은 설계 의도. 악성 플러그인은 이미 임의 코드 실행 가능.
- **제보 불가**: 의도된 기능.

### 27. L-04. 관리자 아바타 업로드 경로 ← 관리자는 신뢰

- **검증**: `management/views.py:369` — 관리자 전용
- **결론**: 관리자 계정은 이미 전체 시스템 제어 가능. 관리자 경로의 검증 차이는 보안 위협이 아님.
- **제보 불가**.

### 28. L-06. CanDeletePost = CanEditPost ← 설계 결정

- **결론**: 의도적인 단순화. 대부분의 포럼에서 편집 가능하면 삭제도 가능.
- **제보 불가**.

### 29. L-07. EditTopicForm.populate_obj ← 이론적

- **결론**: 현재 폼 필드로는 악용 불가. 향후 필드 추가 시의 잠재적 위험이지만 현재 취약하지 않음.
- **제보 불가**.

### 30. L-08. 하드코딩된 그룹 ID 4 ← 설치 시 고정

- **결론**: FlaskBB의 설치 스크립트가 고정 ID로 그룹을 생성. 정상 사용에서는 문제 없음.
- **제보 불가**.

### 31. L-09. DB 에러 세부정보 노출 ← 매우 제한적

- **결론**: 플러그인 마이그레이션 실패 시에만 발생. 일반 사용자에게 노출되지 않음.
- **제보 불가** (실용적 위협 없음).

### 32. L-10. LDAP Injection docstring ← 코드가 아님

- **결론**: 실행되지 않는 docstring의 예제 코드. 복사-붙여넣기 위험은 있으나 취약점이 아님.
- **제보 불가**.

### 33. I-08. DebugToolbar 무조건 초기화 ← 자체 보호

- **검증**: Flask-DebugToolbar는 `DEBUG=False`일 때 스스로 비활성화
- **결론**: `DEBUG=True`로 프로덕션 배포하는 것 자체가 문제. DebugToolbar의 방어 동작은 정상.
- **제보 불가**.

### 34. I-09. Celery 브로커 인증 없음 ← 기본 개발 설정

- **결론**: 기본 설정의 `localhost` Redis는 개발용. 프로덕션 배포 시 변경해야 할 설정.
- **제보 불가** (문서 개선 요청은 가능).

---

## 제보 권장 요약

### 높은 우선순위 (즉시 제보 권장) — 9건

| # | ID | 제목 | CWE | 공격 난이도 |
|---|-----|------|-----|------------|
| 1 | **M-05** | BlockTooManyFailedLogins 미등록 (dead code) | CWE-307 | 낮음 |
| 2 | **M-16** | 첨부파일 접근 제어 부재 | CWE-862 | 낮음 |
| 3 | **M-19** | PostgreSQL 사용자명 대소문자 우회 | CWE-178 | 낮음 |
| 4 | **M-03** | GET 로그아웃 CSRF | CWE-352 | 낮음 |
| 5 | **M-04** | 패스워드 리셋 토큰 재사용 | CWE-613 | 중간 |
| 6 | **M-12** | 이메일 열거 | CWE-204 | 낮음 |
| 7 | **M-14** | 비인증 마크다운 프리뷰 DoS | CWE-400 | 낮음 |
| 8 | **L-14** | Host Header Poisoning | CWE-644 | 중간 |
| 9 | **M-01** | LIKE 와일드카드 인젝션 | CWE-943 | 낮음 |

### 중간 우선순위 (조건부 제보 가능) — 4건

| # | ID | 제목 | 조건 |
|---|-----|------|------|
| 10 | **C-01** | 하드코딩된 SECRET_KEY | 미변경 배포 시 |
| 11 | **H-01** | SVG XSS | 관리자가 svg 허용 시 |
| 12 | **M-13** | SimpleCache 권한 캐시 | 멀티워커 배포 시 |
| 13 | **M-15** | PIL JPEG/jpg 불일치 | 기능 버그 |

### 강화 요청 (보안 개선 제안) — 10건

| # | ID | 제목 |
|---|-----|------|
| 14 | M-02 | LIKE 인젝션 (관리자) |
| 15 | M-08 | 보안 헤더 부재 |
| 16 | M-09 | 비원자적 카운터 |
| 17 | M-10 | 로그인 카운터 레이스 |
| 18 | L-01 | Remember Cookie HttpOnly |
| 19 | L-02 | 패스워드 복잡도 |
| 20 | L-03 | URL Scheme http |
| 21 | L-05 | 예측 가능 아바타 파일명 |
| 22 | L-11 | 세션 쿠키 플래그 |
| 23 | L-12 | update_lastseen 성능 |

### 제보 불가 — 11건 (D등급)

M-06, M-07, M-11, L-04, L-06, L-07, L-08, L-09, L-10, I-08, I-09 — 오탐, 설계 의도, 이론적 위험, 또는 현재 코드에서 안전.

---

## 최종 판정

| 구분 | 건수 |
|------|------|
| 중복 제거 | -6건 |
| A등급 (실제 공격 가능, 제보 ✅) | **9건** |
| B등급 (조건부 공격 가능, 제보 ✅) | **4건** |
| C등급 (강화 권장, ⚠️) | **10건** |
| D등급 (제보 불가, ❌) | **11건** |

**총 34개 고유 발견 중 13건이 실제 제보 가능한 보안 취약점 (A+B등급)**

FlaskBB GitHub Issues에 제보 시 권장 순서:
1. **M-05 + M-10** (계정 잠금 미작동 + 카운터 레이스) — 함께 보고
2. **M-16** (첨부파일 접근 제어) — 개발자 인지 상태이므로 PR 환영 가능
3. **M-19** (PostgreSQL 대소문자) — 명확한 코드 버그, 수정 간단
4. **M-03** (GET 로그아웃 CSRF) — 표준적 취약점
5. **M-04** (토큰 재사용) — 인증 취약점
6. **M-12** (이메일 열거) — 일반적이지만 수정 간단
7. **M-14** (마크다운 DoS) — `login_required` 추가로 해결
8. **L-14 + C-01** (Host Header + SECRET_KEY) — 설정 강화
9. **M-01** (LIKE 인젝션) — `escape_like()` 사용으로 해결

---

*본 재검토는 FlaskBB 소스코드(2026-09-29 기준)를 직접 대조하여 수행. 각 발견 사항의 정확한 코드 위치와 실제 동작을 검증함.*
