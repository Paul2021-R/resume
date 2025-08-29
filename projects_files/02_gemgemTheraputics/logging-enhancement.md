# 로깅 시스템 개선 제안

이 문서는 현재 프로젝트의 로깅 시스템(파일 로깅 및 DB 로깅)을 분석하고, 효과성 및 효율성 측면에서 개선할 수 있는 방안을 제안합니다.

## 현재 로깅 시스템 분석 요약

1.  **Winston을 이용한 파일 및 콘솔 로깅:**
    *   `src/common/module/common-logger.module.ts`에서 Winston을 설정하며, 콘솔 및 일별 파일 로테이션(winstonDaily)을 사용합니다.
    *   콘솔 로그는 `log.level` 설정에 따르며, 파일 로그는 `silly` 레벨로 모든 로그를 기록합니다.
    *   파일은 `LOG_DIR/YYYY-MM` 경로에 `YYYY-MM-DD.log` 형식으로 저장되며, 3시간마다 로테이션되고 최대 20MB 크기, 30일 보관 정책을 가집니다.

2.  **HTTP 요청 DB 로깅 (`HttpLoggingInterceptor`):**
    *   `src/common/interceptors/http-logging.interceptor.ts`는 전역 인터셉터로 HTTP 요청 및 응답 상세 정보를 캡처합니다.
    *   로그는 인메모리 큐에 저장되며, 큐가 `maxLimit`(기본 5개)에 도달하거나 애플리케이션 종료 시 `AgendaService`를 통해 비동기적으로 DB(`prisma.httpLog.createMany`)에 저장됩니다.
    *   특정 URL(`health`, `session/check-health`)은 DB 로깅에서 제외됩니다.

3.  **로거 미들웨어 (`LoggerMiddleware`):**
    *   `src/common/middleware/logger.middleware.ts`는 NestJS 미들웨어로, 기본적인 요청/응답 정보를 콘솔/파일에 로깅합니다.
    *   경로, IP, User-Agent 기반의 스킵 조건이 있습니다.

4.  **초기화 (`main.ts`):**
    *   Winston 로거는 전역 로거로 설정되며, `HttpLoggingInterceptor`는 전역 인터셉터로 등록됩니다. 예외 필터 또한 로깅에 활용됩니다.

## 개선 제안

### 1. 파일 로깅 (`winstonDaily` 빈도 조정)

*   **문제점:** `winstonDaily`의 `frequency: '3h'` 설정은 일반적인 일별 로테이션(`datePattern: 'YYYY-MM-DD'`)과 함께 사용될 때 혼란을 야기하거나 의도치 않은 파일 덮어쓰기를 발생시킬 수 있습니다.
*   **제안:**
    *   **일별 로테이션 유지:** 특별한 이유가 없다면 `frequency` 설정을 제거하거나 `24h` 등으로 명시하여 일별 로테이션을 명확히 합니다.
    *   **시간별 로테이션 필요 시:** 만약 3시간 또는 그 이하의 시간 단위 로테이션이 필요하다면, `datePattern`을 `YYYY-MM-DD-HH` 또는 `YYYY-MM-DD-HH-mm` 등으로 변경하여 파일 이름에 시간 정보를 포함시켜 로그 파일이 덮어쓰여지는 것을 방지합니다.

### 2. DB 로깅 (로그 손실 방지 및 주기적 플러시)

*   **문제점:** 현재 인메모리 큐에 저장된 로그는 `maxLimit`에 도달하거나 애플리케이션이 정상 종료될 때만 DB에 저장됩니다. 예기치 않은 애플리케이션 충돌 시 큐에 남아있는 로그가 손실될 수 있습니다.
*   **제안:**
    *   **주기적 플러시 메커니즘 추가:** `HttpLoggingInterceptor` 내부에 `setInterval` 또는 `AgendaService`를 활용하여 `maxLimit`에 도달하지 않더라도 일정 시간(예: 5초 또는 10초)마다 큐에 있는 로그를 DB에 비동기적으로 저장하는 로직을 추가합니다. 이는 로그 손실 위험을 줄이고, 로그가 더 빠르게 DB에 반영되도록 합니다.
    *   **더 강력한 지속성 고려 (선택 사항):** 로그 손실이 절대적으로 허용되지 않는 중요한 시스템이라면, 인메모리 큐 대신 Redis Streams, Kafka와 같은 메시지 큐나 임시 파일에 먼저 기록하는 방식을 고려할 수 있습니다. (현재 시스템의 복잡성 및 요구사항에 따라 판단)

### 3. `LoggerMiddleware`와 `HttpLoggingInterceptor` 간의 역할 재정의

*   **문제점:** `LoggerMiddleware`와 `HttpLoggingInterceptor` 모두 HTTP 요청/응답 정보를 로깅하며, 일부 정보에서 중복이 발생합니다.
*   **제안:**
    *   **역할 분리 명확화:**
        *   `HttpLoggingInterceptor`: 상세한 HTTP 요청/응답 정보(성능 지표, 사용자 정보, 에러 상세 등)를 DB에 저장하는 역할에 집중합니다. 이는 분석 및 감사 목적에 적합합니다.
        *   `LoggerMiddleware`: 개발 환경에서 콘솔에 빠르게 요청 흐름을 파악할 수 있는 간략한 정보(메서드, URL, 상태 코드, 응답 시간 등)를 로깅하는 역할로 제한하거나, 필요에 따라 제거를 고려합니다. 프로덕션 환경에서는 `HttpLoggingInterceptor`의 DB 로그가 더 유용할 수 있습니다.

### 4. 로깅 일관성 및 민감 정보 마스킹

*   **문제점:** 코드 베이스 내에서 `console.log`의 사용이 발견되었으며, `maskSensitiveInfo()` 함수가 주석 처리되어 있어 민감 정보가 로그에 노출될 위험이 있습니다.
*   **제안:**
    *   **`LOGGER` 인스턴스 사용 강제:** 모든 로깅은 주입된 Winston `LOGGER` 인스턴스를 통해서만 이루어지도록 코딩 표준을 강화합니다. `console.log`는 개발/디버깅 목적으로만 임시로 사용하고 커밋 전 제거하도록 합니다.
    *   **민감 정보 마스킹 구현:** `src/common/module/common-logger.module.ts`에 주석 처리된 `maskSensitiveInfo()` 함수를 구현하고 활성화하여 비밀번호, API 키 등 민감한 정보가 로그에 기록되지 않도록 합니다. 이는 보안상 매우 중요합니다.

### 5. 로그 수준 최적화

*   **문제점:** 파일 로그의 `silly` 레벨은 모든 상세 정보를 기록하여 로그 파일 크기가 매우 커지고, 이는 디스크 공간 및 I/O 성능에 영향을 줄 수 있습니다.
*   **제안:**
    *   **프로덕션 환경 파일 로그 수준 조정:** 프로덕션 환경에서는 파일 로그의 레벨을 `info` 또는 `debug` 등으로 조정하여 필요한 정보만 기록하고 불필요한 상세 로그를 줄입니다. `silly` 레벨은 개발 또는 특정 디버깅 상황에서만 활성화하도록 합니다.

이러한 개선 사항들을 통해 로깅 시스템의 효율성, 신뢰성 및 보안을 향상시킬 수 있습니다.
