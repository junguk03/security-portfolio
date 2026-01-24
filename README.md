# Security Learning Archive

![Days](https://img.shields.io/badge/Days-30+-blue)
![Vulns](https://img.shields.io/badge/Vulnerabilities-0-red)
![Wargames](https://img.shields.io/badge/Wargames-15%2F68-yellow)

보안 학습 내용을 정리하고 관리하는 저장소

---

## 폴더 구조

```
security-portfolio/
├── README.md
├── .gitignore
├── .gitmessage
│
├── chatvas-security/       # chatvas 프로젝트 보안 학습
│   ├── vulnerable/         # 취약한 코드
│   ├── secure/             # 수정된 코드
│   ├── attack-scripts/     # 공격 스크립트
│   ├── reports/            # 취약점 분석 문서
│   ├── screenshots/        # Burp Suite 캡처
│   └── LEARNING_LOG.md
│
├── clashroyale-security/   # ClashRoyale 프로젝트 보안 학습
│   └── ...
│
├── wargames/               # 온라인 워게임 풀이
│   ├── README.md           # 전체 요약
│   └── overthewire/
│       ├── bandit/         # 리눅스 기초 (15/34)
│       │   ├── README.md
│       │   ├── level-00-to-05/
│       │   ├── level-06-to-10/
│       │   └── level-11-to-15/
│       └── natas/          # 웹 보안 (0/34)
│
├── general-security/       # 일반 보안 실습
│   └── ...
│
└── docs/                   # 참고 자료
    └── ...
```

---

## 각 폴더 설명

### chatvas-security

chatvas 프로젝트에서 발견한 보안 취약점을 분석하고 수정하는 과정을 기록.

작업 순서:
1. vulnerable/ - 취약한 코드 작성
2. attack-scripts/ - 공격 스크립트로 검증
3. reports/ - 취약점 분석 문서 작성
4. screenshots/ - Burp Suite로 캡처
5. secure/ - 안전한 코드로 수정

기술 스택: TypeScript, Node.js, Next.js

### clashroyale-security

ClashRoyale 프로젝트에서 발견한 보안 취약점을 분석하고 수정하는 과정을 기록.

GitHub: https://github.com/junguk03/ClashRoyalProject

작업 순서는 chatvas-security와 동일.

### wargames

OverTheWire 등 온라인 워게임 풀이를 기록.

- Bandit: 리눅스 기초 (명령어, SSH, 파일 권한)
- Natas: 웹 취약점 (XSS, SQL Injection, LFI 등)

### general-security

OWASP Top 10 등 일반적인 보안 취약점을 실습.

주요 실습 주제:
- 인증 우회 (Authentication Bypass)
- IDOR (Insecure Direct Object Reference)
- JWT 조작
- Mass Assignment
- Rate Limiting 우회

기술 스택: TypeScript, Python, Bash

### docs

공통 참고 자료 및 도구 사용법 정리.

---

## 사용 도구

### 필수
- Burp Suite Community - 웹 보안 테스트
- Firefox + FoxyProxy - 프록시 설정
- curl, jq - API 테스트
- VSCode - 코드 작성

### 선택
- Postman - API 테스트
- Python - 공격 스크립트 작성
- Docker - 격리된 환경 구성

---

## 커밋 규칙

`.gitmessage` 템플릿 사용 (git commit 시 자동 적용)

커밋 타입:
- `vuln:` - 취약점 발견
- `sec:` - 보안 수정
- `docs:` - 문서 작성
- `wargame:` - 워게임 풀이
- `feat:` - 기능 추가
- `fix:` - 버그 수정

---

## 학습 진행 상황

### 워게임
| 워게임 | 진행 | 상태 |
|--------|------|------|
| [Bandit](./wargames/overthewire/bandit/README.md) | 15/34 | 진행 중 |
| [Natas](./wargames/overthewire/natas/README.md) | 0/34 | 예정 |

### 취약점 실습
| 프로젝트 | 발견 | 해결 |
|----------|------|------|
| chatvas | 0 | 0 |
| clashroyale | 0 | 0 |
| general | 0 | 0 |

진행 계획:
- [x] 프로젝트 구조 설정
- [x] Bandit Level 0-15 풀이
- [ ] Burp Suite 설치 및 설정
- [ ] chatvas 프로젝트 보안 분석 시작
- [ ] Natas 워게임 시작

---

## 시작하기

### Burp Suite 설치

docs/burp-setup.md 참고

### 첫 취약점 실습

```bash
cd general-security/
cat LEARNING_LOG.md
```

### 워게임 시작

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Password: bandit0
```

---

## 주의사항

절대 실제 서비스에 공격하지 말 것. 이 저장소의 모든 내용은 학습 목적으로만 사용.

학습 원칙:
1. 취약점 발견 → 공격 → 분석 → 수정 순서
2. 모든 과정을 문서화
3. Burp Suite로 트래픽 캡처
4. Git으로 진행 상황 추적

---

## 참고 자료

- [OverTheWire Wargames](https://overthewire.org/wargames/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)

추천 학습 순서:
1. OverTheWire Bandit (리눅스 기초)
2. OWASP Top 10 (웹 취약점 기초)
3. OverTheWire Natas (웹 공격 실습)
4. 실제 프로젝트 보안 분석

---

Last Updated: 2026-01-03
