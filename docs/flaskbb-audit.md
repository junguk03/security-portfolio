# FlaskBB 보안 감사 보고서

**대상**: [FlaskBB](https://github.com/flaskbb/flaskbb) (Flask 기반 오픈소스 포럼)
**감사 일자**: 2026-09-28
**감사 방법**: 소스코드 정적 분석 (5개 병렬 감사 에이전트)
**분석 범위**: 인증/인가, SQL Injection, XSS/SSTI, 파일 업로드/SSRF, 로직 버그/레이스 컨디션
**Python 파일 수**: 134개

---

## 요약

| 심각도 | 건수 |
|--------|------|
| 🔴 CRITICAL | 1 |
| 🟠 HIGH | 1 |
| 🟡 MEDIUM | 16 |
| 🟢 LOW | 10 |
| ℹ️ INFO | 7 |

총 **28개 고유 발견** (중복 제거 후), **7개 긍정적 보안 관행** 확인.

FlaskBB는 전반적으로 견고한 보안 아키텍처를 갖추고 있다: SQLAlchemy ORM을 통한 일관된 파라미터화, Jinja2 자동 이스케이프, `secure_filename()` 및 UUID 기반 파일 저장, `send_from_directory()` 사용, `escape=True`로 설정된 mistune 마크다운 렌더러, 견고한 오픈 리다이렉트 방어 등. 그러나 기본 설정의 하드코딩된 시크릿 키, 레이스 컨디션, 누락된 보안 헤더 등 개선이 필요한 영역이 발견되었다.

---

## 목차

1. [CRITICAL 발견](#critical-발견)
2. [HIGH 발견](#high-발견)
3. [MEDIUM 발견](#medium-발견)
4. [LOW 발견](#low-발견)
5. [긍정적 보안 관행](#긍정적-보안-관행)
6. [다음 감사 계획](#다음-감사-계획)
7. [감사 방법론](#감사-방법론)

---

## CRITICAL 발견

### C-01. 하드코딩된 기본 SECRET_KEY 및 WTF_CSRF_SECRET_KEY

- **분류**: OWASP A04 Cryptographic Failures / CWE-798
- **위치**: `configs/default.py:178, 184`
- **CVSS**: 9.8

```python
SECRET_KEY = "secret key"
WTF_CSRF_SECRET_KEY = "reallyhardtoguess"
```

**공격 시나리오**: 관리자가 기본값을 변경하지 않고 배포하면, 공격자가 이 알려진 키로:
1. 세션 쿠키를 위조하여 어떤 사용자(관리자 포함)로든 로그인
2. 패스워드 리셋 JWT 토큰을 생성하여 임의 계정 탈취
3. CSRF 토큰을 위조하여 모든 CSRF 보호 무력화

**수정안**: non-DEBUG 모드에서 기본 SECRET_KEY와 동일한 값이면 시작 거부. `flaskbb install` 시 랜덤 키 생성 강제.

---

## HIGH 발견

### H-01. SVG 첨부파일을 통한 저장형 XSS (관리자 설정 의존)

- **분류**: OWASP A05 Injection / CWE-79
- **위치**: `forum/utils.py:176-188`, `upload/views.py:47-48`
- **CVSS**: 6.1

**트리거 조건**: 관리자가 `ATTACHMENT_TYPES` 설정에 `svg`를 추가한 경우.

```python
as_attachment=not attachment.is_image,  # SVG는 image/* → 인라인 서빙
```

**공격 시나리오**:
1. SVG 파일에 `<script>alert(document.cookie)</script>` 삽입
2. `is_image`가 `image/svg+xml`을 True로 판단 → `as_attachment=False`
3. 브라우저가 SVG 내 JavaScript 실행 → **저장형 XSS**

**수정안**: `image/svg+xml` MIME 타입을 인라인 서빙 차단 목록에 추가. 또는 모든 첨부파일에 `X-Content-Type-Options: nosniff` + `Content-Disposition: attachment`.

---

## MEDIUM 발견

### M-01. LIKE 와일드카드 인젝션 — 검색 작성자 필터

- **분류**: OWASP A05 Injection / CWE-943
- **위치**: `search/service.py:74`

```python
stmt = stmt.where(model.username.ilike(f"%{author}%"))
# escape_like() 미사용, escape="\\" 미전달
```

**공격**: `%` 입력 → 모든 사용자명 매칭. `___` → 3글자 사용자명 열거.
**수정안**: `escape_like(author)` + `escape="\\"` 사용 (동일 코드베이스 `user/views.py:280`에서 올바르게 사용 중).

### M-02. LIKE 와일드카드 인젝션 — 관리자 첨부파일 검색

- **분류**: OWASP A05 Injection / CWE-943
- **위치**: `management/forms.py:114`

```python
pattern = "%{}%".format(self.search_query.data or "")
```

관리자 전용이라 영향 제한적이나, `escape_like()` 미사용은 동일한 문제.

### M-03. GET 요청으로 로그아웃 — CSRF 로그아웃 공격

- **분류**: OWASP A01 Broken Access Control / CWE-352
- **위치**: `auth/views.py:77-83`

```python
class Logout(MethodView):
    def get(self):  # GET으로 상태 변경
        logout_user()
```

**공격**: `<img src="https://forum.example.com/auth/logout">` → 피해자 자동 로그아웃.
**수정안**: POST로 변경 + CSRF 토큰 검증.

### M-04. 패스워드 리셋 토큰 재사용

- **분류**: OWASP A07 Authentication Failures / CWE-613
- **위치**: `auth/services/password.py:55-66`

토큰에 nonce나 패스워드 해시 바인딩이 없어, 1시간 만료 기간 내 동일 토큰으로 반복 리셋 가능.
**수정안**: 현재 패스워드 해시를 JWT payload에 포함, 변경 시 자동 무효화.

### M-05. BlockTooManyFailedLogins 미등록 — 무제한 Brute Force

- **분류**: OWASP A07 Authentication Failures / CWE-307
- **위치**: `auth/plugins.py`

`MarkFailedLogin`은 등록되어 실패 횟수 카운트는 하지만, `BlockTooManyFailedLogins`가 `flaskbb_authenticate` 훅에 등록되지 않아 잠금 검사가 실행되지 않는다. IP 기반 rate limit만 존재하여 다중 IP 공격에 무방비.

### M-06. Rate Limiting 체크 주석 처리

- **분류**: OWASP A07 Authentication Failures / CWE-799
- **위치**: `auth/views.py:427-433`

```python
def check_rate_limiting():
    # TODO: Figure this out
    # return limiter.check()
```

DB 설정 기반 rate limit이 비활성화. 관리자 패널에서 설정을 변경해도 실제 동작하지 않음.

### M-07. 토큰 만료 시간 계산 시점 오류

- **분류**: OWASP A07 Authentication Failures / CWE-613
- **위치**: `tokens/serializer.py:47`

`__init__`에서 `now() + timedelta` 계산 → `dumps()` 호출 시가 아닌 인스턴스 생성 시점 기준. 현재는 요청마다 새 인스턴스 생성하여 안전하지만, 캐싱 시 취약.

### M-08. CSP 및 보안 헤더 전면 부재

- **분류**: OWASP A02 Security Misconfiguration / CWE-693
- **위치**: 전체 코드베이스

`Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options` 헤더가 어디에도 설정되지 않아, XSS 발견 시 방어 계층 부재. 클릭재킹에도 취약.

### M-09. 비원자적 카운터 증가 — Post/Topic/Forum 카운트

- **분류**: CWE-362 (Race Condition)
- **위치**: `forum/models.py:421-425, 971`

```python
user.post_count += 1      # Python 레벨 read-modify-write
topic.post_count += 1     # 동시 요청 시 lost update
topic.forum.post_count += 1
```

**트리거**: 동일 토픽에 2명이 동시 포스팅 → 카운트 1개 손실. 지속 공격으로 카운트 조작 가능.
**수정안**: `user.post_count = User.post_count + 1` (SQL 레벨 atomic).

### M-10. 비원자적 로그인 시도 카운터

- **분류**: CWE-362
- **위치**: `auth/services/authentication.py:~160`

```python
user.login_attempts += 1  # 병렬 요청 시 lost update → 잠금 임계값 우회
```

Brute force 방어 약화. M-05와 결합 시 계정 잠금 완전 무력화.

### M-11. 플러그인 훅 출력 Markup 인젝션

- **분류**: OWASP A05 Injection / CWE-79
- **위치**: `plugins/utils.py:127`, `templates/_macros/navigation.html:23`

```python
return Markup(result)  # 플러그인 반환값을 무조건 safe로 렌더링
```

악성 또는 침해된 플러그인 설치 시 모든 페이지에 임의 JavaScript 삽입 가능.

### M-12. 이메일 열거 — 패스워드 리셋

- **분류**: OWASP A07 Authentication Failures / CWE-204
- **위치**: `auth/views.py:218-236`

유효 이메일: "Email sent!" + 리다이렉트 / 무효 이메일: 에러 메시지 + 폼 재렌더링.
**수정안**: 항상 동일 메시지 + 리다이렉트.

### M-13. 밴 후 캐시된 권한으로 지속 접근

- **분류**: CWE-613
- **위치**: `user/models.py:381-401`

`@cache.memoize()` 사용. 멀티 워커 환경에서 `SimpleCache`는 프로세스 로컬 → 밴 처리한 워커 외 다른 워커에서 60초간 기존 권한 유지.

### M-14. 비인증 마크다운 프리뷰 엔드포인트 — DoS

- **분류**: OWASP A02 Security Misconfiguration / CWE-400
- **위치**: `forum/views.py:1125-1132`

인증 없이 최대 32MB 마크다운 렌더링 요청 가능 → CPU/메모리 소모.
**수정안**: `login_required` 추가 + 콘텐츠 크기 제한 (64KB).

### M-15. 아바타 PIL 검증 JPEG/jpg 불일치

- **분류**: CWE-345 (Insufficient Verification)
- **위치**: `utils/uploads.py:211-218`

PIL은 `"jpeg"` 반환, `AVATAR_EXTENSIONS`는 `["jpg"]` → 비교 항상 실패. PIL 기반 콘텐츠 타입 검증이 사실상 무력화되어 확장자 검증만 작동.

### M-16. 첨부파일 접근 제어 부재

- **분류**: OWASP A01 Broken Access Control / CWE-862
- **위치**: `upload/views.py:34-50`

비공개 포럼의 첨부파일도 URL만 알면 인증 없이 접근 가능. 코드 주석에도 `"Attachments are not yet access control checked"` 명시.

---

## LOW 발견

### L-01. REMEMBER_COOKIE_HTTPONLY = False

- **위치**: `configs/default.py:220`
- XSS 시 remember-me 쿠키 탈취 가능. 프로덕션 템플릿은 True이나 기본값이 False.

### L-02. 패스워드 복잡도 미검증

- **위치**: `auth/forms.py:74-79`
- 빈 문자열이 아닌 모든 패스워드 허용. 길이/복잡도 요구사항 없음.

### L-03. PREFERRED_URL_SCHEME 기본값 http

- **위치**: `configs/default.py:45`
- 패스워드 리셋 링크 등 외부 URL이 `http://`로 생성 → 토큰 평문 노출.

### L-04. 관리자 아바타 업로드 검증 경로 불일치

- **위치**: `management/views.py:367-370`
- 관리자 경로는 `DefaultAvatarUpdateHandler` 파이프라인을 거치지 않아 일반 사용자와 다른 검증 적용.

### L-05. 예측 가능한 아바타 파일명

- **위치**: `utils/uploads.py:34-38`
- `avatar_{username}.{ext}` → 열거 가능. 보안 위협은 낮으나 UUID 기반이 권장.

### L-06. CanDeletePost = CanEditPost 동일 권한

- **위치**: `utils/requirements.py:282`
- 편집 권한과 삭제 권한이 분리되지 않음.

### L-07. EditTopicForm.populate_obj 다중 객체 적용

- **위치**: `forum/forms.py:141-148`
- 폼 필드가 topic과 post 객체 양쪽에 적용. 향후 필드 추가 시 의도하지 않은 속성 설정 위험.

### L-08. 하드코딩된 그룹 ID 4 (회원)

- **위치**: `auth/views.py:~180`
- DB 재구성 시 잘못된 그룹 배정 가능.

### L-09. DB 에러 세부정보 노출 — 플러그인 마이그레이션

- **위치**: `app.py:463`
- 플러그인 마이그레이션과 무관한 `OperationalError` 시 raw exception을 re-raise.

### L-10. LDAP Injection — Docstring 예제 코드

- **위치**: `core/auth/authentication.py:77, 222`
- 실행되지 않는 예제이나, 복사-붙여넣기 시 LDAP 인젝션 패턴.

---

## 긍정적 보안 관행

다음 영역은 올바르게 구현되어 있음을 확인:

| 영역 | 상태 |
|------|------|
| **SQL Injection** | SQLAlchemy ORM 일관 사용, 파라미터화 쿼리. FTS5/tsvector 안전 처리 |
| **SSTI** | `render_template_string()` 런타임 미사용. 모든 뷰가 파일 기반 템플릿 사용 |
| **DOM XSS** | `element.append(string)` 텍스트 노드 생성. `innerHTML`은 서버 렌더링 HTML만 사용 |
| **SSRF** | 런타임 URL 페칭 없음 (requests, urllib, httpx 미사용) |
| **역직렬화** | `pickle.loads()` 마이그레이션 스크립트에만 존재, 런타임 JSON 사용 |
| **경로 탐색** | `secure_filename()` + UUID 파일명 + `send_from_directory()` |
| **오픈 리다이렉트** | Django 기반 URL 검증 + `ALLOWED_HOSTS` 화이트리스트 |
| **마크다운 렌더링** | mistune `escape=True` → HTML/JavaScript 주입 차단 |
| **권한 계층** | 관리자 뷰에 일관된 `IsAdmin`/`IsAtleastModerator` 데코레이터 |
| **타이밍 공격 방어** | 미등록 사용자에도 `check_password_hash` 더미 호출 |

---

## 다음 감사 계획

### 2라운드 (다음 주기) — 동적 분석
- [ ] FlaskBB 로컬 인스턴스 구동 후 실제 공격 시도 (Burp Suite / OWASP ZAP)
- [ ] 레이스 컨디션 PoC — 동시 POST 요청으로 카운터 조작 재현
- [ ] 마크다운 렌더러 퍼징 — ReDoS, 메모리 소모 패턴
- [ ] 세션 고정/재사용 시나리오 실제 검증

### 3라운드 — 의존성 및 설정
- [ ] `pip-audit` / `osv-scanner`로 transitive 의존성 취약점 스캔
- [ ] Celery worker (`celery_worker.py`) 보안 검토
- [ ] Docker 설정 (`docker/`) 보안 검토
- [ ] 테스트 커버리지 분석 — 보안 테스트 존재 여부

### 4라운드 — 플러그인 생태계
- [ ] 주요 FlaskBB 플러그인 소스코드 감사
- [ ] 플러그인 격리 메커니즘 검증
- [ ] 플러그인 설치/업데이트 경로의 공급망 공격 벡터

---

## 감사 방법론

### 분석 영역 및 검색 패턴

| 감사 영역 | 주요 검색 패턴 |
|-----------|----------------|
| SQL Injection | `text(`, `execute(`, `.format(`, `f"SELECT`, `ilike(f"`, `literal_column` |
| XSS / SSTI | `\|safe`, `Markup(`, `render_template_string`, `autoescape`, `dangerouslySetInnerHTML` |
| 인증/인가 | `login_required`, `@allows.requires`, `SECRET_KEY`, `jwt`, `token`, `session` |
| 파일 업로드 | `save(`, `send_file`, `send_from_directory`, `secure_filename`, `open(`, `os.path` |
| 로직 버그 | `+= 1`, `count`, `race`, `lock`, `atomic`, `transaction`, `before_request` |

### 분석 파일 (주요)

- `auth/views.py`, `auth/services/*.py`, `auth/plugins.py`
- `forum/views.py`, `forum/models.py`, `forum/forms.py`, `forum/utils.py`
- `management/views.py`, `management/forms.py`
- `search/service.py`, `search/backends/*.py`
- `upload/views.py`, `utils/uploads.py`, `utils/validators.py`
- `plugins/utils.py`, `tokens/serializer.py`
- `configs/default.py`, `app.py`, `markup.py`
- `templates/` (전체 Jinja2 템플릿)

### 확인된 안전 영역

- 2차 SQL Injection: FTS 인덱스에 저장된 사용자 데이터는 쿼리 구문이 아닌 인덱스 콘텐츠로 처리
- DOM XSS: `editor.js`의 `renderUser()`는 텍스트 노드 생성 (`append()`)
- 역직렬화: `pickle.loads()` 마이그레이션 전용, 런타임은 JSON
- 오픈 리다이렉트: 제어 문자 정규화 + 백슬래시 변환 + `ALLOWED_HOSTS` 검증

---

*이 보고서는 정적 소스코드 분석 결과이며, 런타임 동적 테스트는 다음 감사 라운드에서 수행 예정.*
