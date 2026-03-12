# LAMP Stack 로그인/회원가입 프로젝트

## 프로젝트 개요

Zorin OS에 설치된 LAMP 스택(Linux, Apache, MySQL, PHP)을 활용하여 사용자 인증 시스템(로그인/회원가입)을 구현하는 웹 애플리케이션입니다.

---

## 기술 스택

| 구성요소 | 역할 |
|---------|------|
| Linux (Zorin OS) | 운영체제 |
| Apache2 | 웹 서버 |
| MySQL | 데이터베이스 |
| PHP | 서버 사이드 스크립트 |
| HTML/CSS | 프론트엔드 |

---

## 프로젝트 구조

```
/var/www/html/auth/
├── index.php           # 메인 페이지 (로그인 후 리다이렉트)
├── login.php           # 로그인 페이지
├── register.php        # 회원가입 페이지
├── logout.php          # 로그아웃 처리
├── dashboard.php       # 로그인 후 대시보드
├── config/
│   └── db.php          # 데이터베이스 연결 설정
├── includes/
│   ├── auth.php        # 인증 함수 모음
│   └── validate.php    # 유효성 검사 함수
└── assets/
    ├── css/
    │   └── style.css   # 스타일시트
    └── js/
        └── main.js     # 클라이언트 스크립트
```

---

## 데이터베이스 스키마

```sql
CREATE DATABASE auth_db;

CREATE TABLE users (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    username    VARCHAR(50)  NOT NULL UNIQUE,
    email       VARCHAR(100) NOT NULL UNIQUE,
    password    VARCHAR(255) NOT NULL,  -- bcrypt 해시
    created_at  DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at  DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

---

## 플로우 다이어그램

### 1. 전체 시스템 아키텍처

```mermaid
graph TD
    A[사용자 브라우저] -->|HTTP Request| B[Apache2 Web Server]
    B -->|PHP 처리| C[PHP Engine]
    C -->|쿼리| D[(MySQL Database)]
    D -->|결과 반환| C
    C -->|HTML 응답 생성| B
    B -->|HTTP Response| A

    subgraph LAMP Stack on Zorin OS
        B
        C
        D
    end
```

---

### 2. 회원가입 흐름

```mermaid
flowchart TD
    A([시작]) --> B[회원가입 페이지 접속\nregister.php]
    B --> C[폼 입력\n이름 / 이메일 / 비밀번호]
    C --> D{클라이언트\n유효성 검사}
    D -->|실패| E[오류 메시지 표시]
    E --> C
    D -->|통과| F[POST 요청 전송]
    F --> G{서버\n유효성 검사}
    G -->|실패| H[오류 응답 반환]
    H --> C
    G -->|통과| I{이메일/아이디\n중복 확인}
    I -->|중복| J[중복 오류 메시지]
    J --> C
    I -->|사용 가능| K[비밀번호 bcrypt 해싱]
    K --> L[DB에 사용자 저장]
    L --> M[세션 생성]
    M --> N([대시보드로 이동])
```

---

### 3. 로그인 흐름

```mermaid
flowchart TD
    A([시작]) --> B[로그인 페이지 접속\nlogin.php]
    B --> C[이메일 / 비밀번호 입력]
    C --> D[POST 요청 전송]
    D --> E{서버\n유효성 검사}
    E -->|빈 값| F[입력값 오류 메시지]
    F --> C
    E -->|통과| G[DB에서 이메일로\n사용자 조회]
    G --> H{사용자\n존재 여부}
    H -->|없음| I[인증 실패 메시지]
    I --> C
    H -->|있음| J[password_verify\n비밀번호 검증]
    J --> K{일치 여부}
    K -->|불일치| I
    K -->|일치| L[세션 생성\nsession_start]
    L --> M[SESSION 변수 저장\nuser_id, username]
    M --> N([대시보드로 이동\ndashboard.php])
```

---

### 4. 세션 및 로그아웃 흐름

```mermaid
flowchart TD
    A[페이지 접근] --> B{세션 확인\n$_SESSION 유효?}
    B -->|세션 없음| C[로그인 페이지로\n리다이렉트]
    B -->|세션 유효| D[보호된 페이지 표시\ndashboard.php]
    D --> E[로그아웃 버튼 클릭]
    E --> F[logout.php 호출]
    F --> G[session_unset]
    G --> H[session_destroy]
    H --> I[로그인 페이지로\n리다이렉트]
```

---

### 5. 데이터베이스 연동 구조

```mermaid
erDiagram
    USERS {
        int id PK
        varchar username
        varchar email
        varchar password
        datetime created_at
        datetime updated_at
    }
```

---

## 보안 고려사항

- **비밀번호 해싱**: `password_hash()` + bcrypt 알고리즘 사용
- **SQL 인젝션 방지**: PDO Prepared Statements 사용
- **XSS 방지**: `htmlspecialchars()`로 출력값 이스케이프
- **세션 고정 공격 방지**: 로그인 성공 시 `session_regenerate_id(true)` 호출
- **CSRF 방지**: 폼에 토큰 삽입 및 검증

---

## 개발 환경 설정

```bash
# Apache, MySQL, PHP 설치 확인
sudo systemctl status apache2
sudo systemctl status mysql

# 웹 루트 디렉토리 이동
cd /var/www/html

# 프로젝트 디렉토리 생성
sudo mkdir auth
sudo chown -R $USER:$USER /var/www/html/auth
```
