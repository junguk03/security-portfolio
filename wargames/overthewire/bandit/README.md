# OverTheWire Bandit

![Progress](https://img.shields.io/badge/Progress-15%2F34-brightgreen)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-blue)

리눅스 기초 명령어와 보안 개념을 학습하는 워게임

- **URL**: https://overthewire.org/wargames/bandit/
- **난이도**: 초급
- **학습 내용**: 리눅스 기초, 명령어, SSH, 파일 권한

---

## 진행 상황

| 레벨 | 주제 | 상태 |
|------|------|------|
| [Level 0-5](./level-00-to-05/README.md) | 기초 명령어, 파일 시스템 | ✅ 완료 |
| [Level 6-10](./level-06-to-10/README.md) | 파일 검색, 텍스트 처리 | ✅ 완료 |
| [Level 11-15](./level-11-to-15/README.md) | 인코딩, 압축, SSH 키, 네트워크 | ✅ 완료 |
| Level 16-20 | 네트워크, 포트 스캐닝 | ⏳ 진행 예정 |
| Level 21-25 | cron, 스크립팅 | ⏳ 진행 예정 |
| Level 26-30 | git, 권한 상승 | ⏳ 진행 예정 |
| Level 31-34 | 고급 주제 | ⏳ 진행 예정 |

---

## 배운 핵심 명령어 요약

### 파일 시스템
```bash
ls -a          # 숨김 파일 포함 목록
cd             # 디렉토리 이동
cat            # 파일 내용 출력
file           # 파일 타입 확인
find           # 조건에 맞는 파일 검색
```

### 텍스트 처리
```bash
grep           # 패턴 검색
sort           # 정렬
uniq           # 중복 제거
strings        # 바이너리에서 문자열 추출
tr             # 문자 변환
```

### 인코딩/압축
```bash
base64 -d      # base64 디코딩
xxd -r         # hexdump 역변환
gunzip         # gzip 압축 해제
bunzip2        # bzip2 압축 해제
tar -xvf       # tar 아카이브 해제
```

### SSH
```bash
ssh user@host -p port      # 원격 접속
ssh -i keyfile user@host   # 개인키로 접속
```

### 스트림 리다이렉션
```bash
>              # 출력을 파일로
2>/dev/null    # 에러 숨기기
|              # 파이프 (명령어 연결)
```

---

## 시작하기

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Password: bandit0
```

---

## 학습 목표

- [x] 리눅스 기본 명령어 숙달
- [x] SSH 사용법 학습
- [x] 파일 권한 및 소유자 이해
- [ ] 네트워크 기초 (nc, telnet 등)
- [ ] 스크립팅 기초

---

Last Updated: 2026-01-03
