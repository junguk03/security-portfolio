# Wargame Solutions

OverTheWire 등 온라인 워게임 풀이 모음

---

## 워게임 목록

### OverTheWire Bandit
- URL: https://overthewire.org/wargames/bandit/
- 난이도: 초급
- 학습 내용: 리눅스 기초, 명령어, SSH, 파일 권한
- 진행 상황: 5/34 레벨

### OverTheWire Natas
- URL: https://overthewire.org/wargames/natas/
- 난이도: 중급
- 학습 내용: 웹 취약점 (XSS, SQL Injection, LFI 등)
- 진행 상황: 0/34 레벨

---

## Bandit 풀이

### Level 0
SSH 접속 기초

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Password: bandit0
```

### Level 1
readme 파일 읽기

```bash
cat readme
```

### Level 2
특수 문자가 포함된 파일명 처리

```bash
cat ./-
# 또는
cat < -
```

### Level 3
공백이 포함된 파일명 처리

```bash
cat "spaces in this filename"
# 또는
cat spaces\ in\ this\ filename
```

### Level 4
숨겨진 파일 찾기

```bash
cd inhere
ls -a           # 숨겨진 파일 확인
cat .hidden     # 숨김 파일 읽기
```

배운 것:
- `ls -a`: 숨겨진 파일 보기 (.으로 시작하는 파일)
- 리눅스에서 `.`으로 시작하는 파일은 숨김 처리됨
- `cd`로 디렉토리 이동 후 파일 탐색

### Level 5
사람이 읽을 수 있는 파일 찾기 (파일 타입 확인)

```bash
cd inhere
file ./*        # 모든 파일의 타입 확인
cat ./-file07   # ASCII text 파일 읽기
```

배운 것:
- `file`: 파일 타입 확인 명령어
- 여러 파일 중에서 조건에 맞는 파일 찾기
- 와일드카드 `*` 사용법

### Level 6
특정 조건에 맞는 파일 찾기 (크기, 권한)

```bash
cd inhere
find . -type f -size 1033c ! -executable
# 또는
find . -type f -size 1033c -readable ! -executable
cat ./maybehere07/.file2
```

배운 것:
- `find`: 파일 검색 명령어
- `-type f`: 일반 파일만
- `-size 1033c`: 크기가 1033 바이트인 파일
- `! -executable`: 실행 불가능한 파일

---

## Natas 풀이

(아직 시작 안 함)

---

## 학습 목표

### Bandit
- [ ] 리눅스 기본 명령어 숙달
- [ ] SSH 사용법 학습
- [ ] 파일 권한 및 소유자 이해
- [ ] 네트워크 기초 (nc, telnet 등)

### Natas
- [ ] 웹 기초 (HTML, HTTP, Cookie)
- [ ] SQL Injection 기초
- [ ] XSS (Cross-Site Scripting)
- [ ] LFI (Local File Inclusion)
- [ ] Command Injection

---

## 통계

- 총 풀이한 레벨: 5개 (Bandit 0-5)
- 학습 시작일: 2025-12-14
- 다음 목표: Bandit Level 6-10

---

Last Updated: 2025-12-14
