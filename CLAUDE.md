# AI 웹사이트 제작 표준

작성일: 2026-07-31
대상: AI 코드 생성 (Claude Code / Cursor / ChatGPT)
범위: PHP 8.2 + CodeIgniter 4 + MySQL 5.6 + HTML/CSS/JS

> **적용 범위 안내**: 본 문서는 MIB 홈페이지에 **향후 신규로 추가될 PHP 백엔드/관리자 패널(CodeIgniter 4 + MySQL)** 개발에 적용되는 표준입니다.
> 현재 운영 중인 프론트엔드(`public/` 하위 정적 HTML/CSS/JS, GitHub Pages 배포)는 이 표준의 디렉토리 구조·라우팅·DB 규칙 대상이 아니며, 기존 구조를 그대로 유지합니다.
> 신규 백엔드 개발 착수 시 아래 표준을 기준으로 진행해 주세요.

---

## 우선순위 요약

| 순위 | 항목 | 위치 |
| --- | --- | --- |
| 1 | 디렉토리 구조 준수 | 프로젝트 전체 |
| 2 | 네이밍 컨벤션 준수 | 파일·변수·메서드 |
| 3 | 보안 규칙 준수 | DB·입력·출력·파일업로드 |
| 4 | 라우팅 규칙 준수 | `app/Config/Routes.php` |
| 5 | API 응답 포맷 통일 | `app/Controllers/Api/` |
| 6 | 필터 적용 (adminAuth) | `app/Filters/` |
| 7 | DB 마이그레이션 작성 | `app/Database/Migrations/` |
| 8 | .env 설정 분리 | 프로젝트 루트 |
| 9 | 프론트 JS fetch 규격 | `public/js/` |
| 10 | 프로젝트 초기화 파일 동시 생성 | `composer.json`·`spark`·`.env.example` |

---

## 1. 디렉토리 구조

```
프로젝트 루트/
├── app/
│   ├── Config/
│   │   ├── Routes.php
│   │   ├── Database.php
│   │   └── Filters.php
│   ├── Controllers/
│   │   ├── Admin/              ← 백엔드 컨트롤러
│   │   └── Api/                ← API 컨트롤러
│   ├── Models/                 ← 모델
│   ├── Views/
│   │   └── admin/              ← 백엔드 뷰
│   ├── Filters/
│   │   └── AdminAuth.php       ← 로그인 필터
│   └── Database/
│       └── Migrations/         ← 마이그레이션
├── public/                     ← 사이트 루트
│   ├── index.html              ← 프론트 홈
│   ├── css/
│   ├── js/
│   ├── images/
│   ├── uploads/                ← 사용자 업로드
│   └── index.php               ← CI4 진입점
├── writable/                   ← 캐시·로그·세션 (775 권한)
├── system/                     ← composer install 생성
├── vendor/                     ← composer install 생성
├── composer.json               ← AI 생성
├── spark                       ← AI 생성
├── .env.example                ← AI 생성
└── .env                        ← 수동 설정 (저장소 업로드 금지)
```

### 1.1 배치 규칙

| 대상 | 위치 | 비고 |
| --- | --- | --- |
| 프론트 HTML | `public/` 또는 하위 | 루트 기준 상대 경로 사용 |
| CSS | `public/css/` | 인라인 금지 |
| JS | `public/js/` | 인라인 금지 |
| 이미지 | `public/images/` | |
| 업로드 파일 | `public/uploads/` | DB에는 상대 경로만 저장 |
| 백엔드 컨트롤러 | `app/Controllers/Admin/` | |
| API 컨트롤러 | `app/Controllers/Api/` | |
| 모델 | `app/Models/` | |
| 백엔드 뷰 | `app/Views/admin/` | layout.php 공통 레이아웃 필수 |
| 마이그레이션 | `app/Database/Migrations/` | |
| 필터 | `app/Filters/` | |

---

## 2. 네이밍 컨벤션

| 대상 | 규칙 | 예시 |
| --- | --- | --- |
| 컨트롤러 | PascalCase | `Article.php`, `User.php` |
| 모델 | PascalCase + Model 접미사 | `ArticleModel.php` |
| 뷰 | snake_case | `article_list.php`, `login.php` |
| 메서드 | camelCase | `getList()`, `storeData()` |
| 변수 | snake_case | `$article_list`, `$user_name` |
| DB 테이블 | snake_case + 복수형 | `articles`, `admin_users` |
| 라우트 | kebab-case | `/admin/article-list` |
| 네임스페이스 | `App\Controllers\Admin`, `App\Models` | |

---

## 3. 보안 규칙

| 항목 | 규칙 | 위치 |
| --- | --- | --- |
| SQL 인젝션 | Query Builder 또는 파라미터 바인딩 필수. 문자열 연결 SQL 금지 | 모델 전체 |
| XSS | 출력 시 `esc()` 함수 필수 적용 | 뷰 전체 |
| CSRF | 모든 폼에 `csrf_field()` 포함. API는 CSRF 필터 적용 | 폼·API |
| 비밀번호 | `password_hash($pw, PASSWORD_DEFAULT)` 암호화 필수 | 로그인·회원가입 |
| 파일 업로드 | 화이트리스트(jpg/png/pdf), 최대 5MB, 파일명 재생성 | 업로드 로직 |
| 세션 | 데이터베이스 세션 저장 사용 | `app/Config/Session.php` |
| .env | 저장소 업로드 금지. `.env.example`만 업로드 | 프로젝트 루트 |

---

## 4. 라우팅 규칙

```php
// 백엔드: /admin 접두사
$routes->group('admin', ['namespace' => 'App\Controllers\Admin'], function($routes) {
    $routes->get('login', 'Auth::login');
    $routes->post('login', 'Auth::doLogin');
    $routes->get('logout', 'Auth::logout');

    // adminAuth 필터 적용 구간
    $routes->group('', ['filter' => 'adminAuth'], function($routes) {
        $routes->get('dashboard', 'Dashboard::index');
        $routes->get('article', 'Article::index');
        $routes->post('article/store', 'Article::store');
    });
});

// API: /api 접두사
$routes->group('api', ['namespace' => 'App\Controllers\Api'], function($routes) {
    $routes->get('articles', 'Article::list');
    $routes->post('contact', 'Contact::submit');
});
```

---

## 5. 필터

### 5.1 AdminAuth 필터

- 파일: `app/Filters/AdminAuth.php`
- 기능: 세션에 `admin_id` 없으면 `/admin/login` 리다이렉트
- `after()` 메서드는 빈 구현

### 5.2 필터 등록

- 파일: `app/Config/Filters.php`
- `$aliases` 배열에 `'adminAuth' => \App\Filters\AdminAuth::class` 추가

---

## 6. API 응답 포맷

```php
// 성공
return $this->response->setJSON([
    'status'  => true,
    'message' => '操作成功',
    'data'    => $data
]);

// 실패
return $this->response->setStatusCode(400)->setJSON([
    'status'  => false,
    'message' => '操作失败',
    'data'    => null
]);
```

---

## 7. 데이터베이스 규칙

| 항목 | 규칙 |
| --- | --- |
| PK | `id` INT UNSIGNED AUTO_INCREMENT |
| 타임스탬프 | 모든 테이블에 `created_at`, `updated_at` 필수 |
| 소프트 딜리트 | 삭제 기능 테이블에 `deleted_at` 추가, CI4 소프트 딜리트 사용 |
| 외래키 | 논리 외래키만 사용. DB 레벨 외래키 제약 조건 불필요 |
| 인코딩 | `utf8mb4_unicode_ci` |
| 마이그레이션 | `app/Database/Migrations/`에 작성. `php spark migrate` 실행 |

---

## 8. .env 설정

```env
CI_ENVIRONMENT = production
app.baseURL = 'https://도메인.com/'

database.default.hostname = localhost
database.default.database = DB명
database.default.username = 사용자명
database.default.password = 비밀번호
database.default.DBDriver = MySQLi

security.csrfProtection = true
```

- `.env`는 저장소에 업로드하지 않음
- `.env.example`는 저장소에 업로드 (값은 더미)

---

## 9. 프론트 JS fetch 규격

```javascript
fetch('/api/엔드포인트', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-Requested-With': 'XMLHttpRequest'
    },
    body: JSON.stringify(data)
})
.then(res => res.json())
.then(result => {
    if (result.status) {
        // 성공 처리
    } else {
        // 실패 처리: result.message
    }
});
```

---

## 10. 프로젝트 초기화 파일

AI 생성 시 아래 파일도 동시에 생성:

| 파일 | 설명 |
| --- | --- |
| `composer.json` | CI4 의존성 선언 |
| `spark` | CI4 CLI 도구 |
| `.env.example` | 환경 설정 템플릿 |
| `public/index.php` | CI4 진입점 |
| `app/Config/Routes.php` | 라우트 설정 |
| `app/Config/Filters.php` | 필터 등록 |

### 10.1 실행 순서

```bash
composer install              # system/, vendor/ 생성
cp .env.example .env          # .env 복사 후 DB 정보 입력
php spark migrate             # 테이블 생성
chmod -R 775 writable/        # 쓰기 권한
chmod -R 775 public/uploads/  # 쓰기 권한
php spark serve               # 로컬 실행
```
