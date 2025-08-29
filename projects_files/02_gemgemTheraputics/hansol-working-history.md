# 잼잼 API 서버 (gemgem-api) 분석 보고서

## 1. 프로젝트 개요

`gemgem-api` 프로젝트는 **NestJS** 프레임워크를 기반으로 구축된 **TypeScript** 백엔드 애플리케이션입니다. **Prisma ORM**을 사용하여 **MongoDB** 데이터베이스와 상호작용하며, **Redis**를 캐싱 및 세션 관리에 활용하고 있습니다.

주요 목적은 아동의 재활 및 치료 과정을 돕는 게임형 콘텐츠(`gemgem-care`)와 관련된 데이터를 관리하고, 사용자(보호자) 및 아동(플레이어)의 정보를 처리하며, 결제, 알림, 스케줄링 등 다양한 부가 기능을 제공하는 것으로 보입니다.

## 2. 핵심 기술 스택

- **프레임워크**: NestJS (`@nestjs/core`)
- **언어**: TypeScript
- **데이터베이스**: MongoDB with Prisma ORM (`@prisma/client`)
- **캐시/메시지 큐**: Redis (`@liaoliaots/nestjs-redis`), BullMQ (`@nestjs/bullmq`)
- **인증**: JWT (JSON Web Token) 기반 인증 (`@nestjs/jwt`, `passport-jwt`)
- **설정 관리**: `@nestjs/config` (YAML, .env 파일 사용)
- **로깅**: Winston (`nest-winston`)
- **API 문서화**: Swagger (`@nestjs/swagger`)
- **배포/운영**: Docker, Jenkins, Nginx
- **기타 주요 라이브러리**:
    - `axios`: HTTP 클라이언트
    - `agenda`: 잡 스케줄링
    -`mixpanel`: 사용자 행동 분석
    - `notionhq/client`: Notion API 연동
    - `@apple/app-store-server-library`, `google-auth-library`: 인앱 결제 및 인증

## 3. 주요 기능 및 아키텍처 분석

`src/features` 디렉토리를 중심으로 각 기능 모듈이 분리되어 있으며, 이는 유지보수성과 확장성을 높이는 좋은 구조입니다. 주요 기능들은 다음과 같습니다.

### 3.1. 사용자 및 인증 (`users`, `auth`)

- **기능**:
    - 로컬(이메일/비밀번호), 소셜(Google, Kakao, Apple) 로그인 및 회원가입 기능을 제공합니다.
    - JWT를 사용한 인증 상태를 관리하며, `JwtAuthGuard`를 통해 API 접근을 제어합니다.
    - 사용자 정보(이름, 연락처, 자녀 정보 등)를 관리하고, 탈퇴 처리 로직을 포함합니다.
- **DB 모델**: `User`, `Child`
- **특징**: `UserAttribute` 모델을 통해 특정 사용자에게 특별 권한이나 속성(예: 임상시험 참여자)을 부여하는 기능이 있습니다.

### 3.2. 아동 및 치료 관리 (`children`, `therapy-results`, `therapy-schedule`)

- **기능**:
    - 보호자 계정에 여러 명의 자녀를 등록하고 관리할 수 있습니다.
    - 자녀의 재활 치료 게임(`gemgem-care`) 플레이 기록(`PlayRecord`), 치료 결과(`TherapyResult`), 일일 활동 로그(`DailyActivityLog`)를 상세히 저장합니다.
    - `TherapySchedule`을 통해 개인별 맞춤 치료 스케줄(예: `ASAN202409`)을 관리하고, `Agenda`를 이용해 스케줄링된 작업을 처리합니다.
- **DB 모델**: `Child`, `PlayRecord`, `TherapyResult`, `DailyActivityLog`, `TherapySchedule`
- **특징**: `Prisma` 스키마에서 `TherapyHand` (사용 손), `HandClassLabel` (동작 정확도 등급) 등 재활 치료에 특화된 상세한 데이터 타입을 정의하여 전문적인 데이터를 관리합니다.

### 3.3. 게임 콘텐츠 및 설정 관리 (`motions`, `mini-games`, `common-game-settings`)

- **기능**:
    - 치료에 사용되는 손동작(`Motion`) 데이터(가이드 영상, 설명 등)를 관리합니다.
    - 메인 치료 게임 외에 미니게임(`MiniGameResult`)의 결과도 기록합니다.
    - `GameSettings` 모델을 통해 게임의 난이도, 패치노트 등 다양한 설정을 동적으로 관리할 수 있습니다.
- **DB 모델**: `Motion`, `MiniGameResult`, `GameSettings`, `StageConfig`

### 3.4. 결제 시스템 (`payments-system`, `promotions`)

- **기능**:
    - Apple App Store 및 Google Play Store 인앱 결제를 처리하고, 구독 상태를 관리합니다.
    - `PaymentResult`, `PaymentHistory`, `Subscription` 모델을 통해 결제 및 구독 이력을 추적합니다.
    - 프로모션 코드(`Promotion`)를 발급하고 사용 여부를 관리하는 기능이 있습니다.
- **DB 모델**: `PaymentResult`, `Subscription`, `Promotion`, `TherapyTicket`
- **특징**: 결제 관련 로직을 `PaymentsModule`로 통합하여 관리하고 있으며, `TherapyTicket`을 통해 서비스 이용권을 관리하는 것으로 보입니다.

### 3.5. 알림 및 외부 연동 (`alimtalk-notifications`, `notion`, `mixpanel`, `discord`)

- **기능**:
    - NCP SENS를 통해 알림톡(예: 회원가입, 치료 목표 달성)을 발송합니다.
    - `Discord` 웹훅을 이용해 서버의 주요 이벤트(서버 시작, 에러 등)를 개발팀에 알립니다.
    - `Mixpanel`을 사용해 사용자 행동 데이터를 수집 및 분석합니다.
    - `Notion` API를 연동하여 내부 데이터를 노션 페이지와 동기화하거나 리포트를 생성하는 기능(`gemgem-care-report`)이 있을 것으로 추정됩니다.
- **DB 모델**: `AlimtalkNotification`
- **특징**: 다양한 외부 서비스와 적극적으로 연동하여 운영 효율성과 사용자 경험을 높이고 있습니다.

### 3.6. 공통 모듈 및 인프라 (`common`, `main.ts`)

- **기능**:
    - **에러 핸들링**: `AllExceptionsFilter`, `HttpExceptionFilter` 등 여러 단계의 예외 필터를 두어 안정적인 에러 처리를 구현했습니다.
    - **로깅**: `LoggerMiddleware`, `HttpLoggingInterceptor`를 통해 모든 요청과 응답, 주요 이벤트들을 `Winston`으로 기록하고, `httpReqResLogging` 컬렉션에 저장합니다.
    - **요청 제한**: `ThrottlerModule`을 사용해 비정상적인 트래픽으로부터 서버를 보호합니다.
    - **설정 관리**: `configuration.ts`와 `.yaml` 파일을 통해 환경별(local, dev, prod) 설정을 체계적으로 관리합니다.
    - **컴파일러 전환**: `swc`와 `tsc` 컴파일러를 전환하는 스크립트(`switch-compiler.sh`)를 통해 개발 및 빌드 속도를 최적화합니다.

## 4. 배포 및 운영 (DevOps)

- **Docker**: `docker-compose.*.yml` 파일을 통해 각 환경(local, dev, staging, prod)에 맞는 `NestJS`, `Nginx`, `MongoDB`, `Redis` 컨테이너 환경을 구성합니다.
- **Jenkins**: `jenkinsfile`을 통해 CI/CD 파이프라인을 구축하여 테스트, 빌드, 배포 과정을 자동화합니다.
- **Husky**: `pre-commit`, `pre-push` 훅을 사용하여 코드 커밋 전에 린팅(`eslint`)과 포맷팅(`prettier`)을 강제하여 코드 품질을 일관되게 유지합니다.
- **Nginx**: 웹 서버 및 리버스 프록시로 활용하여 정적 파일 서빙과 API 요청 분배를 처리합니다.

## 5. 종합 평가

이 프로젝트는 **NestJS의 모듈 기반 아키텍처를 매우 잘 활용**하여 기능별로 책임과 관심사를 명확하게 분리한 우수한 구조를 가지고 있습니다. 특히 다음과 같은 점에서 높은 수준의 개발 역량을 엿볼 수 있습니다.

- **체계적인 프로젝트 구조**: 기능별 모듈화, 환경별 설정 분리, 공통 기능 추상화가 잘 되어 있어 유지보수와 확장이 용이합니다.
- **안정성 및 신뢰성**: 상세한 로깅, 다단계 예외 처리, 요청 제한 등 안정적인 서비스 운영을 위한 장치들이 잘 마련되어 있습니다.
- **DevOps 자동화**: Docker, Jenkins, Husky를 활용한 CI/CD 및 개발 환경 자동화는 개발 생산성을 크게 향상시킵니다.
- **상세한 데이터 모델링**: `Prisma`를 통해 재활 치료라는 특정 도메인의 요구사항을 매우 상세하고 체계적으로 데이터 모델에 반영했습니다.

**결론적으로, 이 프로젝트는 아동 재활 치료라는 전문 분야의 서비스를 안정적이고 확장 가능하게 구축하기 위해 최신 백엔드 기술과 개발 방법론을 효과적으로 적용한 모범적인 사례라고 할 수 있습니다.**

---

## 6. DevOps 프로세스 개선 작업 내역

### 6.1. Jenkins CI/CD 파이프라인용 배포 스크립트 통합 및 개선

- **목표**: 기존의 대화형, 개별 서비스(NestJS, Nginx) 배포 스크립트를 Jenkins 자동화 환경에 최적화된 단일 스크립트로 통합하고 안정성을 강화합니다.
- **수행 과정**:
    1.  `tag_and_push_nestjs.sh`와 `tag_and_push_nginx.sh` 스크립트의 기능을 분석하여 중복 로직(ECR 로그인, 이미지 태깅 및 푸시)을 식별했습니다.
    2.  모든 사용자 입력을 제거하고, `환경(ENV)`, `서비스별 버전`, `Target 태그 여부`를 명령행 인자로 받아 처리하도록 설계했습니다.
    3.  반복되는 이미지 푸시 로직을 `push_image` 함수로 모듈화하여 코드 재사용성을 높이고 가독성을 개선했습니다.
    4.  NestJS 버전은 `auto` 옵션을 통해 설정 파일에서 동적으로 읽어오거나, 특정 버전을 직접 명시할 수 있도록 유연성을 추가했습니다.
    5.  특정 서비스의 배포를 건너뛸 수 있는 `skip` 옵션을 추가하여 파이프라인의 선택적 실행을 가능하게 했습니다.
    6.  `aws ecr put-image`를 사용하여 `target` 태그를 갱신하는 방식으로 기존 로직을 개선하여, 더 효율적이고 원자적인 태그 관리가 가능하도록 했습니다.
- **결과물**: `docker/cicd-script/tag_and_push_jenkins.sh`

### 6.2. 신규 스크립트 자체 검토 및 잠재적 개선점 도출

- **목표**: 작성된 `tag_and_push_jenkins.sh` 스크립트의 안정성과 견고함을 검증하고, 발생 가능한 엣지 케이스에 대비합니다.
- **검토 내용**:
    - **오류 처리**: 모든 주요 명령어 실행 후 종료 코드를 확인하여, 실패 시 즉시 스크립트를 중단시키는 로직의 적절성을 검증했습니다.
    - **코드 구조**: 함수화를 통한 책임 분리 및 로직의 명확성을 긍정적으로 평가했습니다.
    - **CI/CD 적합성**: 파라미터 기반 실행 방식이 자동화 환경에 매우 적합함을 확인했습니다.
- **도출된 개선점**:
    - **버전 감지 로직**: 현재 버전 감지 로직이 특정 따옴표 형식에 의존하는 문제를 발견했습니다. 다양한 형식(`'1.2.3'`, `"1.2.3"`, `1.2.3`)을 모두 처리할 수 있도록 정규표현식을 강화할 것을 제안했습니다.
    - **실행 경로 독립성**: 스크립트가 프로젝트 루트에서 실행된다는 암묵적인 가정 대신, 스크립트 파일의 위치를 기준으로 경로를 계산하여 어떤 위치에서 실행되어도 안정적으로 동작하도록 개선할 것을 제안했습니다.

이러한 준비 및 개선 과정을 통해, 단순한 코드 작성을 넘어 **안정적이고 확장 가능한 DevOps 프로세스를 구축**하는 데 기여했습니다.
