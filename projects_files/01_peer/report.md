# Peer-Backend 서버 프로그램 분석 보고서

## 1. 구조 (Structure)

본 백엔드 애플리케이션은 Java 언어와 Spring Boot 프레임워크를 기반으로 개발되었습니다. 전형적인 계층형 아키텍처(Layered Architecture)를 따르며, 주요 구성 요소는 다음과 같습니다.

*   **애플리케이션 진입점:** `BackendApplication.java`
*   **데이터베이스:**
    *   MongoDB: `MongoConfig.java` 및 `mongo/` 패키지를 통해 NoSQL 데이터베이스를 활용합니다.
    *   Redis: `RedisRepositoryConfig.java`를 통해 캐싱, 세션 관리 등에 Redis를 활용합니다.
*   **API 계층:** `controller/` 패키지 아래에 RESTful API 엔드포인트들이 정의되어 있습니다.
*   **비즈니스 로직 계층:** `service/` 패키지 아래에 핵심 비즈니스 로직이 구현되어 있습니다.
*   **데이터 접근 계층:** `repository/` 패키지 아래에 데이터베이스와의 상호작용을 담당하는 인터페이스들이 정의되어 있습니다.
*   **데이터 모델:**
    *   `entity/`: 데이터베이스 엔티티 정의.
    *   `dto/`: 데이터 전송 객체(Data Transfer Object) 정의.
*   **보안:**
    *   `config/SecurityConfig.java`, `config/jwt/JwtSecurityConfig.java`: Spring Security를 활용한 인증(Authentication) 및 인가(Authorization) 처리.
    *   `oauth/`: OAuth2 연동 관련 로직.
*   **횡단 관심사 (Cross-cutting Concerns):**
    *   `aspect/`: AOP(Aspect-Oriented Programming)를 활용하여 로깅(`RequestLoggingAspect.java`), 권한 체크(`AuthorCheckAspect.java`), 사용자/팀 트래킹(`UserTrackingAspect.java`, `TeamTrackingAspect.java`) 등을 분리하여 관리합니다.
    *   `annotation/`: 커스텀 어노테이션을 통해 유효성 검사(`CustomSize.java`, `ValidDateTime.java` 등) 및 특정 로직(`AuthorCheck.java`, `NoLogging.java`)을 적용합니다.
*   **유틸리티 및 설정:**
    *   `config/`: 다양한 애플리케이션 설정(`CorsSourceConfig.java`, `SwaggerConfig.java`, `SocketIoConfig.java` 등).
    *   `converter/`: Enum 변환 등 데이터 형식 변환 로직.
    *   `validator/`: 데이터 유효성 검사 로직.
*   **실시간 통신:** `SocketIoConfig.java` 및 `controller/socket/`을 통해 Socket.IO를 활용한 실시간 통신 기능을 제공합니다.
*   **배치 및 스케줄링:** `batch/JobConfig.java` 및 `scheduler/`를 통해 배치 처리 및 주기적인 작업을 수행합니다.
*   **외부 연동:** `private-resources/firebase/`를 통해 Firebase와 연동하여 푸시 알림 등의 기능을 지원할 수 있습니다.
*   **컨테이너화 (Containerization):**
    *   `Dockerfile-dev`, `Dockerfile-prod`, `Dockerfile-test`: 개발, 운영, 테스트 환경별로 애플리케이션을 컨테이너화하기 위한 Dockerfile이 분리되어 있습니다. 이는 각 환경에 최적화된 이미지 빌드를 가능하게 합니다.
    *   `docker-compose.yml`: 다중 컨테이너 애플리케이션(예: 애플리케이션, 데이터베이스, Redis 등)의 정의 및 실행을 관리하여 개발 및 배포 환경 설정을 용이하게 합니다.
*   **CI/CD:** `.github/workflows/`에 `dev_ci_cd.yml`, `main_ci_cd.yml`, `test_ci_cd.yml`, `sonar_cloud_inspector.yml` 등 GitHub Actions 워크플로우가 구성되어 있어 자동화된 빌드, 테스트, 배포 및 코드 품질 검사를 수행합니다.
*   **환경 설정:** `private-resources/` 내에 `application-dev.yml`, `application-prod.yml` 등 환경별 설정 파일이 분리되어 있습니다.

## 2. 긍정적 측면 (Positives)

*   **명확한 계층형 아키텍처:** `controller`, `service`, `repository`, `entity`, `dto` 등으로 패키지가 잘 분리되어 있어 코드의 응집도(Cohesion)를 높이고 결합도(Coupling)를 낮춥니다. 이는 코드의 가독성, 유지보수성 및 확장성을 향상시킵니다.
*   **다양한 기술 스택의 적절한 활용:** MongoDB(NoSQL), Redis(캐싱/세션), JWT/OAuth2(보안), Socket.IO(실시간 통신) 등 현대적인 백엔드 기술 스택을 적절히 조합하여 애플리케이션의 기능적 요구사항과 성능을 동시에 고려하고 있습니다.
*   **AOP를 통한 횡단 관심사 분리:** 로깅, 권한 검사, 트래킹과 같은 횡단 관심사(Cross-cutting Concerns)를 AOP로 분리하여 핵심 비즈니스 로직의 오염을 방지하고, 코드 중복을 줄여 개발 효율성을 높였습니다.
*   **체계적인 유효성 검사 및 데이터 변환:** `validator` 및 `converter` 패키지를 통해 입력 데이터의 유효성 검사와 형식 변환 로직을 분리하여 관리함으로써, 데이터 무결성을 확보하고 코드의 재사용성을 높였습니다.
*   **강력한 CI/CD 파이프라인 구축:** GitHub Actions를 활용한 CI/CD 파이프라인은 개발, 테스트, 배포 과정을 자동화하여 개발 생산성을 크게 향상시키고, SonarCloud 통합을 통해 지속적인 코드 품질 관리가 가능합니다.
*   **환경별 설정 관리의 용이성:** `private-resources/` 내의 다양한 `application-*.yml` 파일을 통해 개발, 테스트, 운영 등 각 환경에 맞는 설정을 유연하게 관리할 수 있습니다.
*   **Firebase 연동:** Firebase Admin SDK를 통해 푸시 알림 등 모바일/웹 클라이언트와의 연동 기능을 확장할 수 있는 기반이 마련되어 있습니다.
*   **Docker를 통한 환경 일관성 및 배포 용이성:** 개발, 테스트, 운영 환경별 Dockerfile과 `docker-compose.yml`을 통해 애플리케이션의 빌드 및 배포 과정을 표준화하고, 환경 간 일관성을 유지하여 개발 및 운영 효율성을 높입니다.

## 3. 부정적 측면 (Negatives)

*   **잠재적 모놀리식(Monolithic) 구조의 한계:** 현재 디렉토리 구조만으로는 마이크로서비스 아키텍처인지 명확히 판단하기 어렵지만, `BackendApplication.java`라는 단일 진입점으로 미루어 볼 때 모놀리식 애플리케이션일 가능성이 높습니다. 서비스 규모가 커질 경우, 단일 애플리케이션은 유지보수 및 기능 확장에 병목 현상을 일으킬 수 있습니다.
*   **NoSQL (MongoDB) 의존성:** MongoDB는 유연한 스키마와 확장성을 제공하지만, 복잡한 관계형 데이터 모델이나 강력한 트랜잭션 일관성(Transactional Consistency)이 요구되는 비즈니스 로직에는 적합하지 않을 수 있습니다. 이러한 경우 데이터 모델링 및 일관성 관리에 추가적인 노력이 필요합니다.
*   **DTO 및 Entity의 잠재적 중복 및 관리 복잡성:** `dto`와 `entity` 패키지 내에 유사한 목적의 클래스들이 많을 수 있으며, 이는 불필요한 매핑 코드의 증가와 코드 중복을 야기할 수 있습니다. (실제 코드 확인 필요)
*   **`private-resources`의 위치 및 보안:** `.env` 파일이나 민감한 설정 파일이 `private-resources` 폴더에 존재하며, 이는 `.gitignore`에 의해 관리되더라도, 프로젝트 루트에 가까이 위치하여 실수로 민감 정보가 커밋될 위험이 존재합니다.
*   **테스트 커버리지의 불확실성:** `src/test/java/peer/backend/` 아래에 일부 패키지(`board`, `config`, `File`, `profile`, `service`, `team`)에만 테스트 코드가 존재하는 것으로 보아, 전체 코드베이스에 대한 테스트 커버리지가 충분하지 않을 수 있습니다. 이는 코드 변경 시 회귀(Regression) 위험을 증가시킬 수 있습니다.

## 4. 개선 포인트 (Improvement Points)

*   **모듈화 또는 마이크로서비스 아키텍처로의 점진적 전환 고려:**
    *   서비스 도메인별로 모듈을 분리하거나, 핵심 비즈니스 도메인을 중심으로 마이크로서비스 아키텍처로의 점진적 전환을 계획하여 애플리케이션의 확장성과 유지보수성을 확보해야 합니다. 이는 장기적인 관점에서 개발 속도와 안정성을 높일 수 있습니다.
*   **데이터베이스 전략 재검토 및 하이브리드 접근:**
    *   복잡한 관계형 데이터가 필요한 도메인에 대해서는 관계형 데이터베이스(RDB) 도입을 고려하거나, MongoDB 사용 시 데이터 모델링을 더욱 정교하게 하고 트랜잭션 및 일관성 관리 전략을 강화해야 합니다. 필요에 따라 Polyglot Persistence(다중 데이터베이스 사용) 전략을 채택할 수 있습니다.
*   **DTO 및 Entity 관리 자동화 및 최적화:**
    *   MapStruct와 같은 매핑 라이브러리를 활용하여 DTO와 Entity 간의 변환 로직을 자동화하고, 불필요한 DTO 생성을 지양하여 코드 중복을 최소화해야 합니다. 이를 통해 개발 생산성을 높이고 유지보수 비용을 절감할 수 있습니다.
*   **보안 강화 및 민감 정보 관리 개선:**
    *   API 키, 데이터베이스 자격 증명 등 민감한 정보는 환경 변수나 Secret Management System(예: HashiCorp Vault, AWS Secrets Manager, Azure Key Vault)을 통해 안전하게 관리하고, `private-resources` 폴더는 빌드/배포 파이프라인에서만 접근하도록 엄격하게 통제해야 합니다.
*   **테스트 커버리지 확대 및 테스트 전략 강화:**
    *   핵심 비즈니스 로직 및 중요 기능에 대한 단위 테스트(Unit Test)와 통합 테스트(Integration Test)를 강화하여 코드 변경에 대한 안정성을 확보해야 합니다. 특히, `service` 계층의 비즈니스 로직에 대한 테스트를 우선적으로 확대하고, API 계층에 대한 통합 테스트도 보강해야 합니다.
*   **API 문서화의 상세화 및 자동화:**
    *   Swagger가 적용되어 있지만, API 명세의 상세화 및 최신화 노력을 통해 프론트엔드 개발자와의 협업 효율을 증대시켜야 합니다. API 변경 시 문서가 자동으로 업데이트되도록 CI/CD 파이프라인에 통합하는 것을 고려할 수 있습니다.
*   **성능 최적화 및 모니터링 시스템 구축:**
    *   Redis를 캐싱 목적으로 활용하는 것 외에, 애플리케이션의 병목 현상(Bottleneck)이 발생하는 부분을 프로파일링(Profiling)하여 쿼리 최적화, 비동기 처리 도입, 스레드 풀 관리 등 성능 개선 작업을 수행해야 합니다.
    *   ELK Stack(Elasticsearch, Logstash, Kibana) 또는 Prometheus/Grafana와 같은 중앙 집중식 로깅 및 모니터링 시스템을 구축하여 운영 환경에서의 문제 진단 및 성능 추이 분석을 용이하게 해야 합니다. 이는 장애 발생 시 빠른 대응과 선제적인 성능 관리에 필수적.
*   **코드 컨벤션 및 정적 분석 도구 활용 강화:**
    *   SonarCloud가 통합되어 있지만, 팀 내 코드 컨벤션을 명확히 정의하고, Checkstyle, PMD 등 추가적인 정적 분석 도구를 CI/CD 파이프라인에 통합하여 코드 품질을 지속적으로 관리해야 합니다.