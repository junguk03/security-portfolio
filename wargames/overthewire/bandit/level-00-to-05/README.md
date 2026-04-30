# Bandit Level 0-5

기초 리눅스 명령어 및 파일 시스템 탐색

---

## Level 0
**SSH 접속 기초**

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Password: bandit0
```

배운 것:
- `ssh`: Secure Shell, 원격 서버 접속 명령어
- `-p`: 포트 지정 옵션

---

## Level 1
**readme 파일 읽기**

```bash
cat readme
```

배운 것:
- `cat`: 파일 내용 출력 명령어

---

## Level 2
**특수 문자가 포함된 파일명 처리**

```bash
cat ./-
# 또는
cat < -
```

배운 것:
- `-`는 표준 입력을 의미하므로 `./`를 붙여 파일로 인식시킴
- `<`: 입력 리다이렉션

---

## Level 3
**공백이 포함된 파일명 처리**

```bash
cat "spaces in this filename"
# 또는
cat spaces\ in\ this\ filename
```

배운 것:
- 따옴표 또는 백슬래시로 공백 처리
- 파일명에 특수문자가 있을 때 이스케이프 처리

---

## Level 4
**숨겨진 파일 찾기**

```bash
cd inhere
ls -a           # 숨겨진 파일 확인
cat .hidden     # 숨김 파일 읽기
```

배운 것:
- `ls -a`: 숨겨진 파일 보기 (`.`으로 시작하는 파일)
- 리눅스에서 `.`으로 시작하는 파일은 숨김 처리됨
- `cd`로 디렉토리 이동 후 파일 탐색

---

## Level 5
**사람이 읽을 수 있는 파일 찾기 (파일 타입 확인)**

```bash
cd inhere
file ./*        # 모든 파일의 타입 확인
cat ./-file07   # ASCII text 파일 읽기
```

배운 것:
- `file`: 파일 타입 확인 명령어
- 여러 파일 중에서 조건에 맞는 파일 찾기
- 와일드카드 `*` 사용법

---

## 이 레벨에서 배운 핵심 명령어

| 명령어 | 설명 |
|--------|------|
| `ssh` | 원격 서버 접속 |
| `cat` | 파일 내용 출력 |
| `ls -a` | 숨김 파일 포함 목록 |
| `cd` | 디렉토리 이동 |
| `file` | 파일 타입 확인 |

---

[← 이전](../README.md) | [다음 →](../level-06-to-10/README.md)
