# Security Portfolio

![Vulns Found](https://img.shields.io/badge/Vulnerabilities_Found-12-red)
![PoC Exploits](https://img.shields.io/badge/PoC_Exploits-9-orange)
![Patches](https://img.shields.io/badge/Fix_Patches-9-green)
![Wargames](https://img.shields.io/badge/Wargames-19%2F68-blue)

보안 연구, 취약점 분석, 워게임 풀이를 정리하는 포트폴리오 저장소

---

## 주요 성과

### FlaskBB 3.0 보안 감사 (2026-09)

오픈소스 Flask 포럼 [FlaskBB](https://github.com/flaskbb/flaskbb) 3.0에서 **9개 보안 취약점** 발견 및 Responsible Disclosure 진행 중

| # | 심각도 | CWE | 취약점 | PoC | 패치 |
|---|--------|-----|--------|-----|------|
| 1 | High | CWE-307 | 로그인 잠금 미작동 (데드코드) | [poc_m05](exploits/flaskbb/poc_m05_bruteforce.py) | [patch](exploits/flaskbb/patches/fix_m05_bruteforce.patch) |
| 2 | High | CWE-644 | Host Header Poisoning (비밀번호 재설정) | [poc_l14](exploits/flaskbb/poc_l14_host_header_poison.py) | [patch](exploits/flaskbb/patches/fix_l14_host_poisoning.patch) |
| 3 | Medium | CWE-613 | 비밀번호 재설정 토큰 재사용 | [poc_m04](exploits/flaskbb/poc_m04_token_reuse.py) | [patch](exploits/flaskbb/patches/fix_m04_token_reuse.patch) |
| 4 | Medium | CWE-352 | GET 로그아웃 CSRF | [poc_m03](exploits/flaskbb/poc_m03_csrf_logout.py) | [patch](exploits/flaskbb/patches/fix_m03_csrf_logout.patch) |
| 5 | Medium | CWE-204 | 이메일 열거 (응답 차이) | [poc_m12](exploits/flaskbb/poc_m12_email_enum.py) | [patch](exploits/flaskbb/patches/fix_m12_email_enum.patch) |
| 6 | Medium | CWE-400 | 비인증 마크다운 프리뷰 DoS | [poc_m14](exploits/flaskbb/poc_m14_markdown_dos.py) | [patch](exploits/flaskbb/patches/fix_m14_markdown_dos.patch) |
| 7 | Medium | CWE-862 | 첨부파일 접근제어 미흡 | [poc_m16](exploits/flaskbb/poc_m16_attachment_bypass.py) | [patch](exploits/flaskbb/patches/fix_m16_attachment_acl.patch) |
| 8 | Low | CWE-943 | LIKE 와일드카드 인젝션 | [poc_m01](exploits/flaskbb/poc_m01_like_injection.py) | [patch](exploits/flaskbb/patches/fix_m01_like_injection.patch) |
| 9 | Low | CWE-178 | 대소문자 사용자명 우회 (PostgreSQL) | [poc_m19](exploits/flaskbb/poc_m19_case_bypass.py) | [patch](exploits/flaskbb/patches/fix_m19_case_bypass.patch) |

- 9개 전체 PoC 스크립트 작성, 6개 실제 인스턴스에서 검증 완료
- 9개 수정 패치 작성 완료
- Responsible Disclosure 이메일 초안 작성 → [DISCLOSURE.md](exploits/flaskbb/DISCLOSURE.md)

### Planner 앱 보안 감사 (2026-04 ~ 05)

Vercel 배포 중인 Next.js 플래너 앱에서 **3개 취약점** 발견, 2개 해결

| 심각도 | 취약점 | CWE | 상태 |
|--------|--------|-----|------|
| High | npm 패키지 취약점 7개 (Next.js HTTP smuggling 포함) | CWE-1104 | ✅ 해결 |
| Medium | 보안 헤더 7개 미설정 | CWE-693 | ✅ 해결 |
| Medium | IDOR — DB 쿼리 user_id 필터 누락 | CWE-639 | ⏳ 미해결 |

- 상세 분석: [SECURITY_AUDIT.md](planner_security/SECURITY_AUDIT.md)
- 해결 기록: [CASES.md](planner_security/CASES.md)

---

## 폴더 구조

```
security-portfolio/
├── exploits/                    # 취약점 익스플로잇
│   └── flaskbb/                 # FlaskBB 3.0 보안 감사
│       ├── poc_*.py             # PoC 스크립트 9개
│       ├── patches/             # 수정 패치 9개
│       └── DISCLOSURE.md        # 제보 이메일 + PR 설명
│
├── docs/                        # 보안 참고 자료
│   ├── security.md              # OWASP 2025 보안 검사 규칙
│   ├── flaskbb-audit.md         # FlaskBB 감사 보고서
│   ├── flaskbb-audit-review.md  # FlaskBB 제보 가능성 재검토
│   ├── owasp-top10.md           # OWASP Top 10 정리
│   ├── jwt-basics.md            # JWT 기초
│   ├── burp-setup.md            # Burp Suite 설정
│   └── tools-guide.md           # 보안 도구 가이드
│
├── planner_security/            # Planner 앱 보안 감사
│   ├── SECURITY_AUDIT.md        # 감사 결과
│   └── CASES.md                 # 해결 사례
│
├── wargames/                    # 워게임 풀이
│   ├── overthewire/
│   │   ├── bandit/              # 리눅스 기초 (18/34)
│   │   └── natas/               # 웹 보안 (예정)
│   └── dreamhack/
│       └── web/                 # Image Storage 등 (1문제)
│
├── web-security-academy/        # PortSwigger (예정)
├── general-security/            # 일반 보안 실습
└── .github/workflows/           # CI/CD
    └── daily-contribution.yml   # 매일 자동 잔디
```

---

## 워게임

| 플랫폼 | 진행 | 주제 | 링크 |
|--------|------|------|------|
| OverTheWire Bandit | 18/34 | 리눅스 기초, SSH, 파일 권한 | [풀이](wargames/overthewire/bandit/README.md) |
| Dreamhack | 1/? | 웹 해킹 (파일 업로드 우회) | [풀이](wargames/dreamhack/README.md) |
| OverTheWire Natas | 0/34 | 웹 보안 | [예정](wargames/overthewire/natas/README.md) |
| PortSwigger Academy | 0/? | 웹 취약점 랩 | [예정](web-security-academy/README.md) |

---

## 보안 분석 방법론

### 정적 분석 체크리스트

[security.md](docs/security.md) — OWASP 2025 기반 16개 검사 항목

CSRF, XSS, SQLi, SSRF, IDOR, Command Injection, Path Traversal, 파일 업로드, 세션/쿠키, 암호화 등

### 스택별 추가 규칙

Next.js, Supabase, Express, Django, Flask, Spring Boot, Go, PHP 커버

### 사용 도구

| 도구 | 용도 |
|------|------|
| Python (requests) | PoC 익스플로잇 작성 |
| Burp Suite | 웹 프록시, 트래픽 캡처 |
| semgrep | SAST 정적 분석 |
| nmap | 포트 스캐닝 |
| Docker | 취약점 검증 환경 구축 |
| gitleaks | 시크릿 스캔 |

---

## 진행 계획

- [x] 프로젝트 구조 설정
- [x] Bandit Level 0-18 풀이
- [x] Dreamhack 웹 해킹 시작
- [x] FlaskBB 보안 감사 (9개 취약점, PoC, 패치)
- [x] Planner 앱 보안 감사 (3개 취약점)
- [x] OWASP 2025 보안 검사 규칙 정리
- [ ] FlaskBB Responsible Disclosure 제보
- [ ] Natas 워게임 시작
- [ ] Web Security Academy 시작
- [ ] 추가 오픈소스 보안 감사

---

## 커밋 규칙

| 타입 | 내용 |
|------|------|
| `exploit:` | PoC / 익스플로잇 |
| `vuln:` | 취약점 발견 |
| `sec:` | 보안 수정 / 패치 |
| `docs:` | 문서 작성 |
| `ci:` | CI/CD 설정 |
| `feat:` | 기능 추가 |
| `fix:` | 버그 수정 |

---

## 주의사항

이 저장소의 모든 익스플로잇과 PoC는 **학습 및 Responsible Disclosure 목적**으로만 작성되었습니다. 허가 없이 실제 서비스에 사용하지 마십시오.

---

Last Updated: 2026-09-29
