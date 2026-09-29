[![Typing SVG](https://readme-typing-svg.demolab.com?font=Fira+Code&size=24&pause=1000&color=38BDF8&width=700&lines=Vulnerability+Researcher+%7C+Exploit+Developer;Parser+Differential+%26+Server-Side+Attack;Web+Exploitation+%7C+Cryptographic+Attack)](https://git.io/typing-svg)

웹 애플리케이션 **취약점 분석 및 익스플로잇 개발**을 하고 있습니다.
서버사이드 로직 결함, 파서 차이(Parser Differential), 암호 프로토콜 공격 등
자동화 도구로 탐지하기 어려운 취약점을 코드 리뷰와 수동 분석으로 찾는 것을 목표로 합니다.

## 🎯 Focus

**Offensive Security — 취약점 분석 & 익스플로잇 개발**

서버사이드 취약점 중에서도 **파서 차이(Parser Differential)** 를 전문 분야로 잡고 있습니다.
HTTP Request Smuggling, URL 파서 불일치, Content-Type 혼동, 인코딩/Collation 차이 등
두 컴포넌트가 같은 입력을 다르게 해석할 때 발생하는 구조적 취약점을 연구합니다.

## 🔐 Vulnerability Research

### 오픈소스 보안 감사

| 프로젝트 | 취약점 | PoC | 패치 | 상태 |
|----------|--------|-----|------|------|
| [FlaskBB 3.0](https://github.com/junguk03/security-portfolio/tree/main/exploits/flaskbb) | 9개 (High 2 / Medium 5 / Low 2) | 9개 | 9개 | Responsible Disclosure 진행 중 |

> CWE-307, CWE-644, CWE-613, CWE-352, CWE-204, CWE-400, CWE-862, CWE-943, CWE-178

### 익스플로잇 개발 경험

- **AES-CBC Padding Oracle Attack** — 서버 에러 응답 차이를 이용해 키 없이 암호문 복호화, Python 자동화 스크립트 작성 (4,800+ 요청, 타임아웃 재시도 로직 포함)
- **Multi-Stage Attack Chain** — PHP 타입 저글링(`===` vs `switch ==`) → Blind RCE(`escapeshellcmd` 우회) → 외부 웹쉘 배포(`curl -o`) → FLAG 실행파일 추출
- **Blind SQL Injection 자동화** — `substr()` 기반 admin 비밀번호 추출 Python 익스플로잇
- **UNION SQL Injection** — `information_schema`를 이용한 DB 구조 탐색 및 숨겨진 테이블 데이터 추출
- **CBC Bit-Flipping** — AES-CBC 암호문 조작을 통한 인증 토큰 변조 및 구조적 한계 분석
- **Parser Differential** — SVG 업로드 시 XML 파서와 HTML 파서의 태그 처리 차이를 이용한 XSS
- **NoSQL Injection** — CouchDB HTTP API 특수 엔드포인트 악용, `undefined === undefined` 인증 우회
- **SSRF / SSTI / Command Injection** — 소켓 레벨 SSRF, Flask SSTI, 셸 메타문자 우회

### 학습 방향

- **일상**: Dreamhack 웹 워게임
- **주간**: CTF 문제 풀이 + 리버싱 기초 (Ghidra)
- **월간**: CVE 분석 + 오픈소스 코드 리뷰
- **전문 분야**: Parser Differential / Protocol-Level 취약점 연구

## 💼 Projects

### 🔹 안심 계약 가디언 (GuardUp) — 한이음 드림업 2026

- **역할**: 팀장 / RAG 파이프라인 · 벡터 검색 · 위험 스코링 설계
- **내용**: 자립준비청년 대상 AI 임대차계약 분석 모바일 앱
- **기술**: Flutter, FastAPI, LangChain, ChromaDB, GPT-4o, CLOVA OCR
- **보안**: Privacy-First 설계, 민감정보 마스킹, 토큰 기반 인증

### 🔹 이전 개발 프로젝트

- **Clash Royale 전적 검색** — AWS EC2, REST API
- **AI Design → Code Converter** — Vercel 서버리스
- **Suchat 랜덤 채팅** — 실시간 WebSocket

> 백엔드 개발 경험이 취약점 분석의 기반이 되고 있습니다.
> 서버 구조를 이해하고 있기에 "어디서 입력이 파싱되고, 어디서 검증이 빠지는지"를 찾을 수 있습니다.

## 🛠 Skills

### Offensive Security

![Python](https://img.shields.io/badge/Python_(Exploit_Dev)-3776AB?logo=python&logoColor=white&style=flat-square)
![Kali Linux](https://img.shields.io/badge/Kali_Linux-557C94?logo=kalilinux&logoColor=white&style=flat-square)
![Burp Suite](https://img.shields.io/badge/Burp_Suite-FF6633?style=flat-square)
![Ghidra](https://img.shields.io/badge/Ghidra_(Reversing)-red?style=flat-square)
![Wireshark](https://img.shields.io/badge/Wireshark-1679A7?logo=wireshark&logoColor=white&style=flat-square)

### Development

![Flask](https://img.shields.io/badge/Flask-000000?logo=flask&logoColor=white&style=flat-square)
![Node.js](https://img.shields.io/badge/Node.js-339933?logo=node.js&logoColor=white&style=flat-square)
![Express](https://img.shields.io/badge/Express-000000?logo=express&logoColor=white&style=flat-square)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?logo=javascript&logoColor=black&style=flat-square)

### Infra

![AWS](https://img.shields.io/badge/AWS_EC2-FF9900?logo=amazonaws&logoColor=white&style=flat-square)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white&style=flat-square)
![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black&style=flat-square)

## 📊 GitHub Stats

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=junguk03&layout=compact&theme=tokyonight)

## 📫 Contact

[![GitHub](https://img.shields.io/badge/GitHub-181717?logo=github&logoColor=white&style=flat-square)](https://github.com/junguk03)

---

> *"코드를 읽고, 구조를 파악하고, 공격 벡터를 도출한다."*
