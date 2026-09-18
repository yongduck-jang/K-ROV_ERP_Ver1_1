# K-ROV_ERP_Ver1_1
K-ROV_ERP_Ver1_1


# 설치 가이드 (CAFE24 리눅스 웹호스팅 기준)

## 0. 준비물
- CAFE24 웹호스팅 계정(FTP 정보)
- MySQL/MariaDB 데이터베이스 1개 (호스팅 관리페이지에서 생성)
- phpMyAdmin 접근 권한
- PHP 8.0 이상, PDO MySQL / mbstring 확장

## 1. 데이터베이스 준비
1. 호스팅 관리페이지 → DB 관리에서 데이터베이스를 생성합니다.
2. phpMyAdmin 접속 → 생성한 DB 선택 → **가져오기(Import)**
3. `sql/schema.sql` 실행 → 이어서 `sql/seed.sql` 실행
   - 순서를 지켜야 합니다. seed는 schema의 테이블을 전제로 합니다.
   - seed에는 역할·권한·메뉴·부서·직급·거래분류·기본 설정이 들어 있습니다.
   - **관리자 계정은 seed에 없습니다.** 3단계 `setup.php`에서 직접 만듭니다.

## 2. 설정 파일 수정
`config/database.php` 를 편집합니다.

```php
define('APP_ENV', 'production');   // 운영 서버는 반드시 production

'production' => [
    'host' => 'localhost',
    'port' => 3306,
    'name' => '발급받은_DB명',
    'user' => '발급받은_DB아이디',
    'pass' => '발급받은_DB비밀번호',
    'charset' => 'utf8mb4',
],
```

> CAFE24는 웹서버와 DB가 같은 호스트이므로 `host`는 `localhost`입니다.
> `APP_ENV`가 `production`이면 오류 메시지가 화면에 출력되지 않고 `logs/php_error.log`에만 기록됩니다.

## 3. 파일 업로드
1. FTP로 `/www/erp/` (또는 원하는 경로)에 전체 파일 업로드
   - 문서 루트 바로 아래(`/www/`)에 올려도 됩니다. 경로는 자동 인식됩니다.
   - 자동 인식이 안 되면 `config/config.php` 의 `$forceBase` 에 `'/erp'` 처럼 직접 지정하세요.
2. 권한 설정
   | 대상 | 권한 |
   |---|---|
   | `uploads/`, `uploads/imports/`, `logs/` | 707 (또는 웹서버 쓰기 가능하도록 755) |
   | `config/` | 설치 중 707 → **설치 완료 후 755로 되돌리기** |
   | 나머지 파일 | 644 / 디렉터리 755 |

## 4. 설치 실행
1. 브라우저에서 `https://도메인/erp/setup.php` 접속
2. 환경 점검표가 모두 **PASS**인지 확인 (FAIL 항목은 안내대로 조치)
3. 회사명 / 관리자 아이디 / 이름 / 비밀번호(영문+숫자 10자 이상) 입력 후 **설치 완료**
4. `config/installed.lock` 파일이 생성되어 재설치가 차단됩니다.

## 5. 설치 후 필수 조치
- [ ] **`setup.php` 파일 삭제** (또는 이름 변경)
- [ ] `config/` 디렉터리 권한을 755로 환원
- [ ] HTTPS 인증서 적용 후 `.htaccess` 하단의 HTTPS 강제 리다이렉트 주석 해제
- [ ] 관리자 로그인 → `관리자 → 환경 설정`에서 회사명·부가세율·세션 시간 확인
- [ ] `관리자 → 사용자 관리`에서 실제 직원 계정 생성, 권한은 최소한으로 부여
- [ ] `금융관리 → 계좌 관리`에서 회사 계좌 등록 후 CSV 업로드 테스트

## 5-1. 기존 v1.0.0 에서 업그레이드하는 경우

1. DB 백업(phpMyAdmin → 내보내기)
2. `sql/upgrade_v1.1.0.sql` 실행 (테이블 2개·권한 5개·메뉴 3개·설정 6개 추가)
3. 전체 파일 덮어쓰기 업로드
4. `uploads/cards/` 디렉터리 생성 및 쓰기 권한(707) 부여 — 파일에 포함된 `.htaccess`도 함께 올립니다.
5. `관리자 → 환경 설정 → 명함 OCR`에서 인식 엔진 선택

## 6. 점검 체크리스트

| 항목 | 확인 방법 | 기대 결과 |
|---|---|---|
| 로그인 | 잘못된 비밀번호 5회 입력 | 계정 일시 잠금 안내 |
| 권한 차단 | 일반직원 계정으로 `/modules/admin/users.php` 직접 입력 | 403 화면 |
| CSRF | 폼 없이 POST 전송 | 419 오류 |
| CSV 업로드 | 같은 파일 2회 업로드 | 두 번째는 전부 "중복" 처리 |
| CSV 다운로드 | Excel로 열기 | 한글 깨짐 없음(BOM 적용) |
| 오류 비노출 | DB 비밀번호를 틀리게 설정 | 상세 오류 대신 안내 문구만 표시 |
| 설치 재실행 | `setup.php` 재접속 | 403 차단 |
| 명함 OCR | 명함 사진 업로드 | 이름·연락처 자동 입력 후 확인 화면 표시 |
| 명함 보안 | `uploads/cards/파일명.jpg` 직접 접근 | 403 차단 (card_image.php 로만 열람) |

## 7. 자주 발생하는 문제

| 증상 | 원인 / 조치 |
|---|---|
| CSS가 깨져 보임 | 경로 자동 인식 실패 → `config/config.php`의 `$forceBase` 직접 지정 |
| "데이터베이스에 연결할 수 없습니다" | `config/database.php` 접속정보 또는 `APP_ENV` 확인 |
| CSV 업로드 시 "파일 업로드 실패" | `uploads/imports/` 권한 확인(707), 파일 5MB 이하인지 확인 |
| 한글이 ???로 저장됨 | DB/테이블 charset이 utf8mb4인지 확인 |
| 그래프가 안 보임 | 외부 CDN 차단 환경 → Chart.js를 내려받아 `assets/js/`에 두고 `includes/footer.php` 경로 수정 |
| 명함이 인식되지 않음 | 브라우저 인식은 정확도 한계가 있음 → 밝은 곳에서 재촬영하거나 CLOVA OCR 사용, 또는 직접 입력 |
| 명함 이미지 업로드 실패 | `uploads/cards/` 권한 707 확인, 이미지 용량(기본 4MB) 확인 |
| CLOVA/Google 호출 실패 | 호스팅 외부 접속(아웃바운드 443) 차단 여부 확인, API 키·Invoke URL 재확인 |
