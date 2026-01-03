# Wargame Solutions

OverTheWire 등 온라인 워게임 풀이 모음

---

## 워게임 목록

### OverTheWire Bandit
- URL: https://overthewire.org/wargames/bandit/
- 난이도: 초급
- 학습 내용: 리눅스 기초, 명령어, SSH, 파일 권한
- 진행 상황: 12/34 레벨

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

### Level 7
서버 전체에서 파일 찾기 (소유자, 그룹, 크기 조건)

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

배운 것:

**경로 개념**
- `./`: 현재 디렉토리에서 시작 (상대 경로)
- `/`: 루트(서버 전체)에서 시작 (절대 경로)
- "somewhere on the server" → `/`에서 검색
- `cat ./file`: 현재 위치의 파일
- `cat /var/lib/file`: 절대 경로로 파일 접근

**find 명령어 옵션**
- `-user bandit7`: 소유자가 bandit7인 파일
- `-group bandit6`: 그룹이 bandit6인 파일
- `-size 33c`: 크기가 33바이트인 파일

**스트림 리다이렉션**
- stdin (0): 표준 입력
- stdout (1): 표준 출력
- stderr (2): 에러 출력
- `2>/dev/null`: 에러 메시지를 /dev/null(휴지통)로 버림
- Permission denied 같은 에러를 숨겨서 가독성 향상

### Level 8
파일에서 특정 단어가 있는 줄 찾기

```bash
grep "millionth" data.txt
```

배운 것:
- `grep`: 파일에서 특정 패턴을 찾는 명령어
- `grep "검색어" 파일명`: 파일 내에서 해당 문자열이 포함된 줄을 출력

### Level 9
중복되지 않은 유일한 줄 찾기

```bash
sort data.txt | uniq -u
```

배운 것:
- `sort`: 파일 내용을 정렬하는 명령어
- `uniq -u`: 중복되지 않은(unique) 줄만 출력
- `|` (파이프): 앞 명령어의 출력을 다음 명령어의 입력으로 전달
- `uniq`는 정렬된 상태에서만 제대로 작동하므로 `sort` 먼저 실행 필요

### Level 10
사람이 읽을 수 있는 문자열 중 특정 패턴으로 시작하는 줄 찾기

```bash
strings data.txt | grep "^="
```

배운 것:
- `strings`: 바이너리 파일에서 사람이 읽을 수 있는 문자열만 추출
- `grep "^="`: `^`는 줄의 시작을 의미하는 정규식 기호
  - `^=`: 줄의 맨 앞에 `=`가 있는 줄만 검색
  - `grep "="` (^없이): `=`가 어디든 포함된 줄 모두 검색

### Level 11
base64로 인코딩된 데이터 디코딩하기

```bash
base64 -d data.txt
```

배운 것:
- `base64`: base64 인코딩/디코딩 명령어
- `-d`: decode 옵션 (디코딩)
- **base64란?**: 바이너리 데이터를 텍스트로 변환하는 인코딩 방식
  - 영문자 + 숫자로만 구성
  - 끝에 `=` 또는 `==`가 붙는 경우가 많음
  - 사람이 읽을 수는 있지만 의미는 알 수 없는 문자열

### Level 12
ROT13 암호 해독하기

```bash
cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"
```

배운 것:
- `tr`: 문자 변환(translate) 명령어
- `"A-Za-z" "N-ZA-Mn-za-m"`: 문자 치환 규칙
  - `A-Z` → `N-ZA-M`: 대문자를 13칸씩 이동 (A→N, B→O, ..., N→A, O→B, ...)
  - `a-z` → `n-za-m`: 소문자도 동일하게 13칸 이동
- **ROT13이란?**: 알파벳을 13칸씩 회전시키는 단순 암호화 방식
  - 같은 방식으로 두 번 적용하면 원문으로 복구됨

### Level 13
(진행 중)

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

- 총 풀이한 레벨: 12개 (Bandit 0-12)
- 학습 시작일: 2025-12-14
- 다음 목표: Bandit Level 13-20

---

Last Updated: 2026-01-03
