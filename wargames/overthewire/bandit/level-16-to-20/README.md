# Bandit Level 16-20

TLS 연결, 포트 스캐닝, SSH 키, 파일 비교

---

## Level 16
**TLS 서버에 직접 연결하여 비밀번호 전송**

```bash
openssl s_client -connect localhost:30001
# 15단계 비밀번호 입력
```

배운 것:

**openssl s_client**
- `openssl`: 암호화/복호화 관련 다목적 도구
- `s_client`: SSL/TLS 서버에 클라이언트로 직접 연결하는 서브커맨드
- `-connect host:port`: 연결할 서버와 포트 지정
- nc (netcat)는 평문 통신, openssl s_client는 암호화된 TLS 통신

**TLS(Transport Layer Security)**
- 네트워크 통신을 암호화하는 프로토콜
- HTTP → HTTPS, 일반 TCP → TLS처럼 평문 통신에 암호화 추가
- 서버 인증서를 통해 서버 신원 확인

---

## Level 17
**nmap 포트 스캔 후 올바른 포트 찾기**

```bash
# 1. 포트 범위 스캔
nmap -p 31000-32000 localhost

# 출력 예시:
# 31046/tcp open unknown
# 31518/tcp open unknown
# 31691/tcp open unknown
# 31790/tcp open unknown  ← 정답
# 31960/tcp open unknown

# 2. 각 포트에 openssl로 연결 테스트
openssl s_client -connect localhost:31790 -quiet
# 16단계 비밀번호 입력 → RSA Private Key 출력

# 3. 키 파일 저장 (임시 디렉토리 생성)
mkdir /tmp/jung
cd /tmp/jung
cat > sshkey.private << EOF
-----BEGIN RSA PRIVATE KEY-----
(출력된 키 붙여넣기)
-----END RSA PRIVATE KEY-----
EOF

# 4. 로컬에서 접속 (localhost → localhost 차단 문제로 로컬에 저장 후 접속)
# 메모장에 키 붙여넣어 .txt로 저장 후:
ssh -i '.\OneDrive\바탕 화면\sshkey.private.txt' -p 2220 bandit17@bandit.labs.overthewire.org

# 5. 비밀번호 파일 찾기
cd /etc/
find -name *bandit17*
cat ./bandit_pass/bandit17
```

배운 것:

**nmap (Network Mapper)**
- 네트워크 탐색과 보안 감사를 위한 핵심 도구
- 보안 분야에서 무조건 알아야 하는 도구
- `-p 31000-32000`: 특정 포트 범위 스캔
- 참고: https://koromoon.blogspot.com/2020/09/nmap.html

**openssl s_client 옵션**
- `-quiet`: 핸드쉐이크 정보 출력 생략 → 서버 응답만 깔끔하게 출력
  - 이 옵션 없으면 RSA Private Key가 제대로 안 보임
  - **중요**: Private Key를 받을 때는 반드시 `-quiet` 필요

**RSA Private Key**
- 비대칭 암호화에서 사용하는 개인키
- `-----BEGIN RSA PRIVATE KEY-----` ~ `-----END RSA PRIVATE KEY-----` 형식
- 이 파일이 있으면 해당 계정으로 SSH 접속 가능

**트러블슈팅**
- 문제: `localhost → localhost` 접속이 최신 환경에서 차단됨
- 해결: Private Key를 로컬 PC에 저장 후 외부에서 SSH로 접속

**파일 이름으로 검색**
- `find -name *bandit17*`: 이름에 bandit17이 포함된 파일 검색
- `*` (와일드카드): 해당 위치에 어떤 문자든 가능

---

## Level 18
**두 파일 비교하여 변경된 줄 찾기**

```bash
# 홈 디렉토리에 passwords.new, passwords.old 존재
ls -al

# 두 파일의 차이점 비교
diff -c passwords.new passwords.old
# ! 표시된 줄이 변경된 부분 → new 파일에 있는 것이 정답
```

배운 것:

**diff 명령어**
- `diff`: 두 파일의 차이점을 비교하는 명령어
- `-c`: context 모드, 변경된 줄 앞뒤 문맥 포함 출력
- 출력 기호 의미:
  - `!`: 해당 줄이 변경됨
  - `+`: 추가된 줄
  - `-`: 삭제된 줄
- `diff A B`에서 `!`는 A에서 B로 바뀐 것이므로 A(new)에 있는 값이 정답

**실무 연결**
- 설정 파일 변경 전후 비교
- 코드 리뷰 시 변경 사항 파악
- 침해 사고 대응 시 정상 파일과 변조 파일 비교

---

## 이 레벨에서 배운 핵심 명령어

| 명령어 | 설명 |
|--------|------|
| `openssl s_client -connect` | TLS 서버 연결 |
| `openssl s_client -quiet` | TLS 연결 (핸드쉐이크 생략) |
| `nmap -p` | 포트 범위 스캔 |
| `diff -c` | 파일 비교 (context 모드) |
| `find -name` | 이름으로 파일 검색 |

---

[← 이전](../level-11-to-15/README.md) | [다음 →](../level-21-to-25/README.md)
