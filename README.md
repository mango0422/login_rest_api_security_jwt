# Login REST API Security JWT

이 프로젝트는 **Spring Boot** 기반의 REST API로, 사용자 인증 및 회원가입 기능을 제공하며 **JWT(JSON Web Token)** 를 활용하여 보안을 강화한 애플리케이션입니다.

## 프로젝트 개요

- **인증 및 회원가입 기능**: 사용자는 `/auth/signup` 엔드포인트를 통해 회원가입을 진행하고, `/auth/login` 엔드포인트를 통해 로그인하여 JWT 토큰을 발급받습니다.
- **JWT 기반 인증**: 로그인 시 발급된 JWT 토큰을 HTTP 요청의 `Authorization` 헤더에 포함시켜 인증된 요청을 수행할 수 있습니다.
- **사용자 정보 조회**: `/users/me` 엔드포인트를 통해 현재 인증된 사용자 정보를, `/users/` 엔드포인트를 통해 모든 사용자 정보를 조회할 수 있습니다.
- **보안 예외 처리**: Spring Security와 JWT 관련 예외를 처리하기 위한 전역 예외 처리(Global Exception Handler)를 구현하였습니다.

## 사용 기술

- **Spring Boot 3.2.5**
- **Spring Security**: 인증 및 인가 처리
- **Spring Data JPA**: 데이터베이스 연동 (MariaDB 사용)
- **JWT (io.jsonwebtoken)**: 토큰 기반 인증
- **Gradle**: 프로젝트 빌드 도구
- **Lombok**: 코드 간결화를 위한 어노테이션 사용

## 프로젝트 구조

```
login_rest_api_security_jwt/
├── src/main/java/profit/login_rest_api_security_jwt
│   ├── config
│   │   ├── ApplicationConfiguration.java   // 스프링 빈 설정 (UserDetailsService, AuthenticationProvider 등)
│   │   ├── JwtAuthenticationFilter.java    // JWT 필터 (요청마다 토큰 검증)
│   │   └── SecurityConfiguration.java        // Spring Security 설정
│   ├── controller
│   │   ├── AuthenticationController.java     // 로그인 및 회원가입 엔드포인트 제공
│   │   └── UserController.java               // 사용자 정보 조회 엔드포인트 제공
│   ├── dto
│   │   ├── LoginUserDto.java                  // 로그인 요청 DTO
│   │   └── RegisterUserDto.java               // 회원가입 요청 DTO
│   ├── entity
│   │   └── User.java                         // 사용자 엔티티 (UserDetails 인터페이스 구현)
│   ├── exceptions
│   │   └── GlobalExceptionHandler.java        // 전역 예외 처리
│   ├── repository
│   │   └── UserRepository.java               // 사용자 데이터 CRUD 처리
│   ├── response
│   │   └── LoginResponse.java                // 로그인 응답 DTO (토큰 및 만료 시간 포함)
│   └── service
│       ├── AuthenticationService.java         // 회원가입 및 로그인 서비스
│       ├── JwtService.java                     // JWT 토큰 생성, 검증 등 서비스
│       └── UserService.java                    // 사용자 관련 서비스 (모든 사용자 조회 등)
├── build.gradle                              // Gradle 빌드 스크립트 (의존성 및 플러그인 설정)
└── application.yaml                          // 애플리케이션 설정 (데이터소스, JWT 시크릿, 포트 등)
```

## 설정 및 실행 방법

### 전제 조건

- **Java 17** 이상
- **Gradle** (프로젝트에 포함된 Gradle Wrapper 사용 가능)
- **MariaDB** 데이터베이스 서버

### 데이터베이스 설정

`application.yaml` 파일에서 데이터베이스 URL, 사용자 이름, 비밀번호 등의 설정을 확인하고 본인의 환경에 맞게 수정합니다.

```yaml
spring:
  datasource:
    url: jdbc:mariadb://[DB_HOST]:[DB_PORT]/[DB_NAME]
    username: [USERNAME]
    password: [PASSWORD]
  jpa:
    hibernate:
      ddl-auto: update
```

### JWT 설정

`application.yaml` 파일에서 JWT 관련 시크릿 키와 만료 시간을 설정합니다.

```yaml
security:
  jwt:
    secret-key: 3cfa76ef14937c1c0ea519f8fc057a80fcd04a7420f8e8bcd0a7567c272e007b
    expiration-time: 3600000 # 밀리초 단위 (예: 3600000 = 1시간)
```

### 실행 방법

1. 프로젝트 루트 디렉토리에서 Gradle 빌드를 실행합니다.

   ```bash
   ./gradlew build
   ```

2. 애플리케이션을 실행합니다.

   ```bash
   ./gradlew bootRun
   ```

3. 서버가 기본적으로 포트 `8005`에서 실행되며, REST API 호출을 통해 기능을 테스트할 수 있습니다.

## API 엔드포인트

### 회원가입

- **URL**: `/auth/signup`
- **Method**: `POST`
- **Request Body 예시**:

  ```json
  {
    "email": "user@example.com",
    "password": "비밀번호",
    "fullName": "홍길동"
  }
  ```

- **Response**: 등록된 사용자 정보

### 로그인

- **URL**: `/auth/login`
- **Method**: `POST`
- **Request Body 예시**:

  ```json
  {
    "email": "user@example.com",
    "password": "비밀번호"
  }
  ```

- **Response**: JWT 토큰과 만료 시간

  ```json
  {
    "token": "발급된JWT토큰",
    "expiresIn": 3600000
  }
  ```

### 현재 사용자 정보 조회

- **URL**: `/users/me`
- **Method**: `GET`
- **설명**: 인증된 사용자의 정보를 반환합니다. 요청 시 JWT 토큰을 포함해야 합니다.

### 전체 사용자 목록 조회

- **URL**: `/users/`
- **Method**: `GET`
- **설명**: 모든 사용자 정보를 반환합니다.

## CORS 설정

CORS는 `SecurityConfiguration` 클래스 내에 설정되어 있으며, 기본적으로 `http://localhost:8005` 도메인에서 오는 `GET`, `POST` 요청과 `Authorization`, `Content-Type` 헤더를 허용합니다.

## 예외 처리

전역 예외 처리(GlobalExceptionHandler)를 통해 인증 및 권한 관련 예외를 비롯한 다양한 예외 상황에 대해 적절한 HTTP 상태 코드와 메시지를 반환합니다.

## 추가 참고 사항

- **보안 강화**: 실제 서비스에서는 JWT 시크릿 키 관리, 토큰 갱신, 추가적인 예외 처리 등을 강화하여 사용하시기 바랍니다.
- **테스트**: Spring Security 테스트를 포함하여 다양한 단위 테스트를 추가할 수 있습니다.
- **관찰성(Observability)**: 예외 발생 시 스택 트레이스를 외부 로깅/관찰성 도구로 전송하는 기능을 추가할 수 있습니다.
