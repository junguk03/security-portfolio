# Bandit Level 6-10

파일 검색, 텍스트 처리, 스트림 리다이렉션

---

## Level 6
**특정 조건에 맞는 파일 찾기 (크기, 권한)**

```bash
cd inhere
find . -type f -size 1033c ! -executable
cat ./maybehere07/.file2
```

배운 것:
- `find`: 파일 검색 명령어
- `-type f`: 일반 파일만
- `-size 1033c`: 크기가 1033 바이트인 파일
- `! -executable`: 실행 불가능한 파일

---

## Level 7
**서버 전체에서 파일 찾기 (소유자, 그룹, 크기 조건)**

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat /var/lib/dpkg/info/bandit7.password
```

배운 것:

**경로 개념**
- `./`: 현재 디렉토리에서 시작 (상대 경로)
- `/`: 루트(서버 전체)에서 시작 (절대 경로)
- "somewhere on the server" → `/`에서 검색

**find 명령어 옵션**
- `-user bandit7`: 소유자가 bandit7인 파일
- `-group bandit6`: 그룹이 bandit6인 파일
- `-size 33c`: 크기가 33바이트인 파일

**스트림 리다이렉션**
- stdin (0): 표준 입력
- stdout (1): 표준 출력
- stderr (2): 에러 출력
- `2>/dev/null`: 에러 메시지를 /dev/null(휴지통)로 버림

---

## Level 8
**파일에서 특정 단어가 있는 줄 찾기**

```bash
grep "millionth" data.txt
```

배운 것:
- `grep`: 파일에서 특정 패턴을 찾는 명령어
- `grep "검색어" 파일명`: 해당 문자열이 포함된 줄을 출력

---

## Level 9
**중복되지 않은 유일한 줄 찾기**

```bash
sort data.txt | uniq -u
```

배운 것:
- `sort`: 파일 내용을 정렬하는 명령어
- `uniq -u`: 중복되지 않은(unique) 줄만 출력
- `|` (파이프): 앞 명령어의 출력을 다음 명령어의 입력으로 전달
- `uniq`는 정렬된 상태에서만 제대로 작동하므로 `sort` 먼저 실행 필요

---

## Level 10
**사람이 읽을 수 있는 문자열 중 특정 패턴으로 시작하는 줄 찾기**

```bash
strings data.txt | grep "^="
```

배운 것:
- `strings`: 바이너리 파일에서 사람이 읽을 수 있는 문자열만 추출
- `grep "^="`: `^`는 줄의 시작을 의미하는 정규식 기호
  - `^=`: 줄의 맨 앞에 `=`가 있는 줄만 검색
  - `grep "="` (^없이): `=`가 어디든 포함된 줄 모두 검색

---

## 이 레벨에서 배운 핵심 명령어

| 명령어 | 설명 |
|--------|------|
| `find` | 조건에 맞는 파일 검색 |
| `grep` | 텍스트 패턴 검색 |
| `sort` | 정렬 |
| `uniq` | 중복 제거 |
| `strings` | 바이너리에서 문자열 추출 |
| `2>/dev/null` | 에러 숨기기 |

---

[← 이전](../level-00-to-05/README.md) | [다음 →](../level-11-to-15/README.md)
