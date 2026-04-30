# Security Learning Archive

![Days](https://img.shields.io/badge/Days-30+-blue)
![Vulns](https://img.shields.io/badge/Vulnerabilities-0-red)
![Wargames](https://img.shields.io/badge/Wargames-18%2F68-yellow)

보안 학습 내용을 정리하고 관리하는 저장소

---

## 폴더 구조

```
security-portfolio/
├── README.md
├── .gitignore
├── .gitmessage
│
├── chatvas-security/         # chatvas 프로젝트 보안 학습
│   ├── vulnerable/
│   ├── secure/
│   ├── attack-scripts/
│   ├── reports/
│   ├── screenshots/
│   └── LEARNING_LOG.md
│
├── clashroyale-security/     # ClashRoyale 프로젝트 보안 학습
│   ├── vulnerable/
│   ├── secure/
│   ├── attack-scripts/
│   ├── reports/
│   ├── screenshots/
│   └── LEARNING_LOG.md
│
├── wargames/                 # 온라인 워게임 풀이
│   ├── README.md
│   ├── overthewire/
│   │   ├── bandit/           # 리눅스 기초 (18/34)
│   │   │   ├── README.md
│   │   │   ├── level-00-to-05/
│   │   │   ├── level-06-to-10/
│   │   │   ├── level-11-to-15/
│   │   │   └── level-16-to-20/
│   │   └── natas/            # 웹 보안 (0/34)
│   └── dreamhack/            # 드림핵 (0/?)
│
├── web-security-academy/     # PortSwigger 웹 보안 아카데미
│   └── README.md
│
├── general-security/         # 일반 보안 실습
│   ├── vulnerable/
│   ├── secure/
│   ├── attack-scripts/
│   ├── reports/
│   └── LEARNING_LOG.md
│
└── docs/                     # 참고 자료
    ├── burp-setup.md
    ├── jwt-basics.md
    ├── owasp-top10.md
    └── tools-guide.md
```

---

## 학습 영역

### 워게임
| 플랫폼 | 진행 | 상태 | 링크 |
|--------|------|------|------|
| OverTheWire Bandit | 18/34 | 진행 중 | [바로가기](./wargames/overthewire/bandit/README.md) |
| OverTheWire Natas | 0/34 | 예정 | [바로가기](./wargames/overthewire/natas/README.md) |
| Dreamhack | 0/? | 예정 | [바로가기](./wargames/dreamhack/README.md) |

### 학습 플랫폼
| 플랫폼 | 진행 | 상태 | 링크 |
|--------|------|------|------|
| PortSwigger Web Security Academy | 0/? | 예정 | [바로가기](./web-security-academy/README.md) |

### 프로젝트 취약점 분석
| 프로젝트 | 발견 | 해결 | 상태 |
|----------|------|------|------|
| chatvas | 0 | 0 | 예정 |
| clashroyale | 0 | 0 | 예정 |
| general | 0 | 0 | 예정 |

---

## 진행 계획

- [x] 프로젝트 구조 설정
- [x] Bandit Level 0-18 풀이
- [ ] Dreamhack 시작
- [ ] Web Security Academy 시작
- [ ] chatvas 프로젝트 보안 분석
- [ ] clashroyale 프로젝트 보안 분석
- [ ] Natas 워게임 시작

---

## 각 영역 설명

### chatvas-security / clashroyale-security

직접 개발한 프로젝트의 취약점을 찾고 분석하는 실습.

작업 순서:
1. `vulnerable/` - 취약한 코드 작성
2. `attack-scripts/` - 공격 스크립트로 검증
3. `reports/` - 취약점 분석 문서 작성 (공격 흐름 다이어그램 포함)
4. `screenshots/` - Burp Suite로 캡처
5. `secure/` - 안전한 코드로 수정

### wargames

- **Bandit**: 리눅스 기초 명령어, SSH, 파일 권한
- **Natas**: 웹 취약점 (XSS, SQLi, LFI 등)
- **Dreamhack**: 웹/시스템/리버싱 전 분야

### web-security-academy

Burp Suite 제작사의 무료 웹 보안 학습 플랫폼.
실습 랩(Lab) 기반으로 체계적인 웹 취약점 학습.

### general-security

OWASP Top 10 기반 일반 취약점 실습:
- 인증 우회 / IDOR / JWT 조작 / SSRF / XSS / SQLi

---

## 사용 도구

| 도구 | 용도 |
|------|------|
| Burp Suite | 웹 보안 테스트, 트래픽 캡처 |
| nmap | 포트 스캐닝, 네트워크 탐색 |
| openssl | TLS 연결, 암호화 분석 |
| netcat (nc) | 네트워크 연결 |
| Firefox + FoxyProxy | 프록시 설정 |
| Python | 공격 스크립트 작성 |

---

## 커밋 규칙

| 타입 | 내용 |
|------|------|
| `docs:` | 문서 작성 |
| `vuln:` | 취약점 발견 |
| `sec:` | 보안 수정 |
| `feat:` | 기능 추가 |
| `fix:` | 버그 수정 |
| `refactor:` | 구조 개선 |

---

## 주의사항

절대 실제 서비스에 공격하지 말 것. 이 저장소의 모든 내용은 학습 목적으로만 사용.

---

## 참고 자료

- [OverTheWire Wargames](https://overthewire.org/wargames/)
- [Dreamhack](https://dreamhack.io)
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

Last Updated: 2026-01-03
