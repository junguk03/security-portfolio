# Wargame Solutions

OverTheWire 등 온라인 워게임 풀이 모음

---

## 워게임 목록

### OverTheWire Bandit
- URL: https://overthewire.org/wargames/bandit/
- 난이도: 초급
- 학습 내용: 리눅스 기초, 명령어, SSH, 파일 권한
- 진행 상황: 6/34 레벨

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

---

## 기타 워게임 풀이

### Image Storage (파일 업로드 취약점)

**취약점 유형:** 파일 업로드 제한 우회 + PHP 웹쉘

**문제 상황:**
이미지 저장 기능이 있는 웹 서버에서 파일 확장자 검증이 미흡하여 `.php` 파일 업로드가 가능한 상황.

**풀이:**

```bash
# 1. PHP 웹쉘 파일 생성
echo '<?php echo file_get_contents("/flag.txt");?>' >> shell1.php
```

업로드된 `shell1.php`에 접근하면 서버가 PHP를 실행하여 `/flag.txt` 내용을 반환 → 플래그 획득.

**핵심 개념:**

- `<?php ... ?>`: PHP 코드 실행 구문
- `file_get_contents("/flag.txt")`: 파일을 문자열로 읽어 반환하는 PHP 내장 함수
- `echo`: 읽은 내용을 HTTP 응답으로 출력

**왜 되는가:**
서버가 업로드된 파일을 정적 파일(이미지)로만 처리해야 하는데, 확장자 검증 없이 저장하면 `.php` 파일도 웹 루트에 올라감. 이후 해당 URL로 접근 시 Apache/Nginx가 PHP 인터프리터로 실행함.

**대응 방법 (방어 관점):**
- 파일 확장자 화이트리스트 검증 (`.jpg`, `.png` 등만 허용)
- MIME 타입 서버 사이드 검증
- 업로드 디렉토리에서 PHP 실행 비활성화

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

- 총 풀이한 문제: 32개 (0단계 11문제, 1단계 20문제, 2단계 1문제)
- 학습 시작일: 2025-12-14
- 다음 목표: Bandit Level 7-10

---

Last Updated: 2025-12-14
