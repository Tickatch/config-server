# Config Server

Tickatch MSA의 **중앙 설정 관리** 서버입니다.

---

## 역할

- 모든 마이크로서비스의 설정 중앙 관리
- Git 저장소 기반 설정 파일 관리
- 환경별 설정 분리 (local, dev, prod)
- 설정 변경 시 동적 갱신 지원

---

## 기술 스택

| 항목 | 버전 |
|------|------|
| Java | 21 |
| Spring Boot | 4.0.0 |
| Spring Cloud | 2025.1.0 |
| Spring Cloud Config | 5.0.x |

---

## 실행 방법

### 로컬 실행

```bash
./gradlew bootRun
```

### Docker 빌드

```bash
./gradlew clean bootJar
docker build -t ghcr.io/tickatch/config-server:latest .
```

---

## 아키텍처

```
┌─────────────────────────────────────────────────┐
│                    GitHub                        │
│  ┌───────────────────────────────────────────┐  │
│  │              config-repo                   │  │
│  │  configs/                                  │  │
│  │  ├── common/application.yml               │  │
│  │  ├── gateway-server/application.yml       │  │
│  │  ├── account-service/application.yml      │  │
│  │  └── ...                                   │  │
│  └───────────────────────────────────────────┘  │
└─────────────────────────────────────────────────┘
                        │
                        ▼
              ┌─────────────────┐
              │  Config Server  │
              │     :8888       │
              └─────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
   ┌─────────┐    ┌─────────┐    ┌─────────┐
   │ Gateway │    │ Account │    │  User   │
   │ Server  │    │ Service │    │ Service │
   └─────────┘    └─────────┘    └─────────┘
```

---

## 포트

| 서비스 | 포트 |
|--------|------|
| config-server | 8888 |

---

## 환경 변수

| 변수 | 기본값 | 설명 |
|------|--------|------|
| `SERVER_PORT` | 8888 | 서버 포트 |
| `CONFIG_REPO_URI` | https://github.com/tickatch/config-repo | Git 저장소 URI |
| `CONFIG_REPO_BRANCH` | main | Git 브랜치 |
| `EUREKA_CLIENT_SERVICEURL_DEFAULTZONE` | http://localhost:8761/eureka/ | Eureka 서버 주소 |
| `ZIPKIN_ENDPOINT` | http://localhost:9411/api/v2/spans | Zipkin 엔드포인트 |

---

## API 엔드포인트

| 엔드포인트 | 설명 |
|------------|------|
| `/{application}/{profile}` | 서비스별 설정 조회 |
| `/{application}/{profile}/{label}` | 특정 브랜치 설정 조회 |
| `/actuator/health` | 헬스 체크 |
| `/actuator/refresh` | 설정 갱신 |

### 예시

```bash
# gateway-server의 local 프로파일 설정 조회
curl http://localhost:8080/gateway-server/local

# account-service의 prod 프로파일 설정 조회
curl http://localhost:8081/account-service/prod
```

---

## 디렉토리 구조

```
config-server/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/tickatch/configserver/
│       │       └── ConfigServerApplication.java
│       └── resources/
│           ├── application.yml
│           └── logback-spring.xml
├── build.gradle
├── settings.gradle
├── Dockerfile
└── README.md
```

---

## 설정 검색 경로

Config Server는 다음 경로에서 설정을 검색합니다:

```yaml
search-paths:
  - configs/common
  - configs/{application}
  - configs/{application}/{profile}
```

---

## 의존 관계

```
Eureka Server (3151, 3152)
        ▲
        │ 등록
        │
Config Server (3100)
```

Config Server는 Eureka Server가 먼저 기동되어야 합니다.