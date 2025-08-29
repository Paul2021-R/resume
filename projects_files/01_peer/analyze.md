# Peer-Backend 프로젝트 분석

이 문서는 `Peer-Backend` 프로젝트에 대해 아무것도 모르는 사람을 대상으로 프로젝트의 역할, 용도, 구조, 그리고 초기 설정 방법을 안내합니다.

## 1. 프로젝트 역할 및 용도

- **Peer-Backend**는 팀 기반 협업 및 소셜 네트워킹 웹 애플리케이션인 **'Peer'의 백엔드 서버**입니다.
- 사용자 인증, 팀 관리, 게시판, 실시간 메시징, 프로필 관리 등 'Peer' 서비스의 핵심 비즈니스 로직과 데이터 처리를 담당합니다.
- 프론트엔드(Peer-Frontend)에 RESTful API를 제공하여 데이터를 주고받습니다.

## 2. 기술 스택 (Technology Stack)

프로젝트를 구성하는 주요 기술은 다음과 같습니다.

- **언어 (Language):** Java 11
- **프레임워크 (Framework):** Spring Boot 2.7.14
- **데이터베이스 (Database):**
    - **MySQL 8:** 주요 데이터 저장소 (팀, 유저, 게시글 등)
    - **MongoDB:** 실시간 알림, 메시지 등 비정형 데이터 저장소
    - **Redis:** 캐싱, JWT 토큰 관리, 실시간 세션 관리
- **인증 (Authentication):** Spring Security, OAuth2, JWT (JSON Web Token)
- **API 문서화 (API Documentation):** Swagger (Springfox)
- **실시간 통신 (Real-time Communication):** Netty-SocketIO
- **빌드 도구 (Build Tool):** Gradle
- **컨테이너화 (Containerization):** Docker, Docker Compose
- **기타:**
    - **Spring Batch:** 배치 처리 작업
    - **Log4j2:** 로깅
    - **Spring Mail:** 이메일 전송

## 3. 프로젝트 구조

프로젝트는 표준적인 Spring Boot의 계층형 아키텍처(Layered Architecture)를 따릅니다.

```
Peer-Backend/
├── .github/              # GitHub Actions 워크플로우, 이슈/PR 템플릿
├── build.gradle          # 프로젝트 의존성 및 빌드 설정
├── docker-compose.yml    # 외부 서비스(DB, Redis 등) 실행 설정
├── Dockerfile-*          # 환경별 Docker 이미지 빌드 파일
├── private-resources/    # (중요) 실제 배포에 사용되는 민감한 설정 파일 (application.yml, secret keys 등)
└── src/
    ├── main/
    │   ├── java/peer/backend/
    │   │   ├── annotation/       # 사용자 정의 어노테이션
    │   │   ├── aspect/           # AOP(관점 지향 프로그래밍) 로직
    │   │   ├── config/           # Security, Swagger, DB 등 핵심 설정
    │   │   ├── controller/       # HTTP 요청을 받는 API 엔드포인트
    │   │   ├── dto/              # 데이터 전송 객체 (Data Transfer Objects)
    │   │   ├── entity/           # JPA 엔티티 (MySQL 테이블과 매핑)
    │   │   ├── exception/        # 예외 처리 핸들러
    │   │   ├── mongo/            # MongoDB 관련 클래스 (도메인, 레포지토리)
    │   │   ├── oauth/            # OAuth2 소셜 로그인 관련 로직
    │   │   ├── repository/       # JPA 레포지토리 (MySQL DB 접근)
    │   │   └── service/          # 비즈니스 로직 구현
    │   └── resources/          # 정적 리소스 및 설정 파일 (개발용)
    └── test/                   # 테스트 코드
```

- **`private-resources`**: 이 디렉토리는 실제 서버 환경에서 사용될 민감한 설정 파일(`application-*.yml`, `firebase-sdk.json` 등)을 관리합니다. `build.gradle`의 `copyGitSubmodule` 태스크를 통해 빌드 시 `src/main/resources`로 복사됩니다. **로컬 개발 환경 설정 시 이 구조를 반드시 이해해야 합니다.**

## 4. 초기 설정 가이드

로컬 환경에서 프로젝트를 실행하기 위한 단계별 가이드입니다.

### 사전 준비물

- **Java 11 (JDK)**
- **Docker** 및 **Docker Compose**

### 설정 단계

1.  **프로젝트 클론**
    ```bash
    git clone https://github.com/peer-42seoul/Peer-Backend.git
    cd Peer-Backend
    ```

2.  **환경 설정 파일 준비 (가장 중요)**
    - `private-resources` 디렉토리에 있는 설정 파일들을 기반으로 로컬 개발용 설정 파일을 만들어야 합니다.
    - `private-resources/application-local.yml` 파일을 `src/main/resources/application-local.yml` 로 복사하거나 새로 생성합니다.
    - `application-local.yml` 파일 내의 데이터베이스, Redis, JWT 시크릿 키 등의 설정값을 자신의 로컬 환경에 맞게 수정합니다.

3.  **외부 서비스 실행 (Docker)**
    - 프로젝트 루트 디렉토리에서 아래 명령어를 실행하여 `docker-compose.yml`에 정의된 MySQL, Redis, MongoDB 컨테이너를 실행합니다.
    ```bash
    docker-compose up -d
    ```
    - 실행 확인: `docker ps` 명령어로 3개의 컨테이너(`peer_backend`, `peer_redis`, `mongodb`)가 정상적으로 실행 중인지 확인합니다.

4.  **프로젝트 빌드**
    - Gradle Wrapper를 사용하여 프로젝트를 빌드합니다. (테스트를 건너뛰려면 `-x test` 옵션을 추가합니다.)
    ```bash
    ./gradlew build -x test
    ```

5.  **애플리케이션 실행**
    - IDE(IntelliJ, Eclipse 등)에서 `src/main/java/peer/backend/BackendApplication.java` 파일을 직접 실행합니다.
    - 또는, 아래 명령어로 실행할 수 있습니다.
    ```bash
    ./gradlew bootRun
    ```

6.  **실행 확인**
    - 애플리케이션이 정상적으로 실행되면, 웹 브라우저에서 Swagger API 문서 페이지에 접속하여 API 목록을 확인합니다.
    - URL: [http://localhost:8080/swagger-ui/index.html](http://localhost:8080/swagger-ui/index.html)

이제 로컬 환경에서 `Peer-Backend` 프로젝트가 성공적으로 실행되었습니다.
