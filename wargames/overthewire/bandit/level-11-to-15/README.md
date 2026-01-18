# Bandit Level 11-15

인코딩, 압축, SSH 키 인증

---

## Level 11
**base64로 인코딩된 데이터 디코딩하기**

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

---

## Level 12
**ROT13 암호 해독하기**

```bash
cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"
```

배운 것:
- `tr`: 문자 변환(translate) 명령어
- `"A-Za-z" "N-ZA-Mn-za-m"`: 문자 치환 규칙
  - `A-Z` → `N-ZA-M`: 대문자를 13칸씩 이동
  - `a-z` → `n-za-m`: 소문자도 동일하게 13칸 이동
- **ROT13이란?**: 알파벳을 13칸씩 회전시키는 단순 암호화 방식
  - 같은 방식으로 두 번 적용하면 원문으로 복구됨

---

## Level 13
**다중 압축 해제 (hexdump → 여러 압축 형식)**

```bash
# 1. 임시 작업 디렉토리 생성
cd /tmp
mktemp -d
# 출력: /tmp/tmp.GtzBy6KtRw

# 2. 생성된 임시 디렉토리로 이동
cd /tmp/tmp.GtzBy6KtRw

# 3. 작업할 파일 복사
cp ~/data.txt .

# 4. hexdump를 바이너리로 복원
xxd -r data.txt > data

# 5. 파일 타입 확인 후 압축 해제 반복
file data

# 압축 타입에 따라 확장자 변경 + 압축 해제
# gzip인 경우:
mv data data.gz && gunzip data.gz

# bzip2인 경우:
mv data data.bz2 && bunzip2 data.bz2

# tar 아카이브인 경우:
tar -xvf data

# 위 과정을 ASCII text가 나올 때까지 반복
cat data8
```

배운 것:

**임시 디렉토리 작업**
- `mktemp -d`: 임시 디렉토리 생성 (안전한 작업 공간)
- `/tmp`: 시스템 임시 파일 저장 위치
- `~/`: 홈 디렉토리 경로

**hexdump 복원**
- `xxd -r`: hexdump를 원본 바이너리로 역변환

**파일 타입 확인 및 압축 해제**
- `file`: 파일의 실제 타입 확인 (확장자와 무관)
- 압축 형식별 명령어:

| 압축 형식 | 확장자 | 해제 명령어 |
|-----------|--------|-------------|
| gzip | .gz | `gunzip` |
| bzip2 | .bz2 | `bunzip2` |
| xz | .xz | `unxz` |
| zip | .zip | `unzip` |
| tar | - | `tar -xvf` |

**tar 명령어 옵션**
- `-x`: extract (압축 풀기)
- `-v`: verbose (처리 과정 출력)
- `-f`: file (파일 이름 지정)

**핵심 개념**
- 파일은 다중으로 압축될 수 있음
- 확장자는 단순 이름일 뿐, `file` 명령어로 실제 타입 확인 필요
- 각 압축 도구는 특정 확장자를 요구하므로 `mv`로 이름 변경 필요

---

## Level 14
**SSH 개인키를 이용한 접속**

```bash
# 로컬에서 개인키 파일을 저장 후 경로 지정하여 접속
ssh -i C:\Users\JungUk\bandit14_key.txt bandit14@bandit.labs.overthewire.org -p 2220
```

배운 것:

**SSH 키 기반 인증**
- `-i`: identity file, 개인키 파일 경로 지정
- 비밀번호 대신 개인키 파일로 인증하는 방식
- 서버의 공개키와 클라이언트의 개인키가 쌍을 이룸

**트러블슈팅**
- 에러: `no authentication methods enabled`
- 원인: 비밀번호 인증이 비활성화된 서버에 비밀번호로 접속 시도
- 해결: 개인키 파일을 로컬에 저장 후 `-i` 옵션으로 경로 지정

**실무 연결**
- 실제 서버 운영 시 비밀번호 인증보다 키 기반 인증이 더 안전
- AWS EC2 등 클라우드 서버도 `.pem` 개인키 파일로 접속
- 개인키 파일은 절대 공유하면 안 됨 (권한도 600으로 제한)

---

## 이 레벨에서 배운 핵심 명령어

| 명령어 | 설명 |
|--------|------|
| `base64 -d` | base64 디코딩 |
| `tr` | 문자 변환 (ROT13 등) |
| `xxd -r` | hexdump 역변환 |
| `mktemp -d` | 임시 디렉토리 생성 |
| `gunzip/bunzip2/tar` | 압축 해제 |
| `ssh -i` | 개인키로 SSH 접속 |

---

[← 이전](../level-06-to-10/README.md) | [다음 →](../level-16-to-20/README.md)
