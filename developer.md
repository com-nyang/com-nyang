# Backend Engineer 프로젝트 이력서 정리

## 핵심 메시지

백엔드 코드를 단순히 구현하는 데서 끝내지 않고, runtime configuration, DB migration, OAuth callback, asset integration, API routing, load test까지 정리해 production 환경에서 실제로 운영 가능한 서비스로 전환했습니다.

## 1. Backend Engineer 이력서 프로젝트 제목 후보

1. Peeple Backend Production Deployment & Runtime Platform 구축
2. AWS EKS 기반 백엔드 서비스 Production 전환
3. 백엔드 API 서버 Production Runtime 및 배포 환경 구축
4. RDS Migration, Runtime Config, API Routing 기반 백엔드 운영 환경 구성
5. Peeple 백엔드 서비스 Production 배포 및 성능 검증

## 2. 한국어 이력서 Bullet

- 로컬/개발 환경 중심의 백엔드 서비스를 AWS EKS 기반 production 환경에서 실행 가능하도록 전환하고, 서비스별 배포 구성, runtime configuration, DB migration, API routing, health check를 정리
- login, main, community, shortform, crew, admin-api 6개 백엔드 서비스를 production namespace `peeple-prd`에 배포하고, 서비스별 Kubernetes Deployment, Service, ServiceAccount를 구성
- 서비스별 Docker image를 AWS ECR에 push하고 Kustomize overlay에서 image tag를 관리하도록 구성하여 production 배포 대상 이미지와 Kubernetes manifest를 일관되게 관리
- 백엔드 runtime config를 코드/이미지에 포함하지 않고 AWS Secrets Manager의 `peeple-prd-env`에서 읽도록 구성하고, `AWS_SECRET_NAME`, `AWS_REGION` 등 production env를 Kubernetes env patch로 주입
- production RDS PostgreSQL 연결을 구성하고, private subnet에 있는 DB에 로컬에서 직접 접근하지 않고 EKS 내부 Kubernetes Job으로 back1, crew, admin migration을 수행하도록 자동화
- migration 중 `public.users` schema 참조 문제를 발견하고 실제 schema 기준에 맞게 수정/재실행하여 `back1-migrate`, `crew-migrate`, `admin-migrate` 완료 상태를 확인
- OAuth/Kakao callback URL을 production domain 기준으로 정리하고 Secrets Manager에 반영해 `https://api.peeplist.com/auth/kakao/callback` 기반 Kakao login이 정상 동작하는 것을 검증
- `api.peeplist.com`과 `admin-api.peeplist.com`을 public API/admin API hostname으로 분리하고, ALB/Gateway API/HTTPRoute 기반 route와 `/healthz` HTTPS health check를 검증
- S3 bucket과 CloudFront public URL을 production asset serving 경로로 구성하고, `assets_public_base_url`을 backend runtime secret에 반영하여 local file system 의존을 줄이는 asset integration 기반을 마련
- API/RDS 부하 테스트 스크립트를 작성하고 `CONCURRENCY=100` 기준 API 38,063건 평균 62.7ms, RDS SELECT 13,560건 실패 0건 결과로 production runtime 상태를 검증

## 3. 백엔드 포지션에 맞는 강한 버전 Bullet

- 6개 백엔드 서비스(login, main, community, shortform, crew, admin-api)를 AWS EKS production namespace에 배포 가능하도록 Docker/ECR/Kustomize/Kubernetes 구성을 정리하고, `/healthz` 기준 서비스 상태 검증 체계를 구성
- DB URL, OAuth/Kakao callback URL, S3/CloudFront asset URL 등 backend runtime dependency를 AWS Secrets Manager 기반 production config로 정리하여 코드/이미지와 환경 설정을 분리
- private RDS PostgreSQL 환경에 맞춰 로컬 직접 접속 대신 EKS 내부 Kubernetes Job 기반 migration 자동화를 구성하고, back1/crew/admin migration을 분리 실행해 schema 반영을 완료
- Gateway API/HTTPRoute 기반으로 public API와 admin API route를 분리하고, `api.peeplist.com`, `admin-api.peeplist.com` HTTPS health check 및 HTTP-to-HTTPS redirect를 검증
- API 38,063건 부하 테스트와 RDS SELECT 13,560건 read test를 수행해 평균 응답시간 62.7ms, 평균 쿼리 시간 3.963ms, DB read 실패 0건을 확인하고 backend runtime 리소스 사용량을 점검

## 4. STAR 형식 상세 설명

### Situation

기존 백엔드 서비스는 로컬/개발 환경 중심으로 실행되고 있었고, production 환경에서 필요한 DB 연결, migration, runtime secret, OAuth callback, asset URL, API routing, health check, 배포 절차가 정리되어 있지 않았습니다. 백엔드 코드를 실제 운영 가능한 서비스로 전환하기 위한 production runtime 구성이 필요했습니다.

### Task

login, main, community, shortform, crew, admin-api 서비스를 AWS EKS 기반 production 환경에 배포 가능하게 만들고, RDS PostgreSQL migration, Secrets Manager 기반 runtime configuration, Kakao OAuth callback, S3/CloudFront asset serving, ALB/Gateway API routing, HTTPS health check, 부하 테스트까지 검증해야 했습니다.

### Action

서비스별 Docker image를 ECR에 push하고 Kustomize overlay에서 image tag를 관리하도록 구성했습니다. 각 서비스의 Kubernetes Deployment, Service, ServiceAccount를 production namespace `peeple-prd`에 배포하고, `/healthz` 기준으로 상태를 확인했습니다. Runtime configuration은 AWS Secrets Manager의 `peeple-prd-env`로 정리하고, Kubernetes env에는 secret value 대신 `AWS_SECRET_NAME`, `AWS_REGION` 등 조회에 필요한 값만 주입했습니다.

RDS PostgreSQL은 private subnet에 구성되어 있어, 로컬 직접 접속 대신 EKS 내부 Kubernetes Job을 통해 migration을 수행하도록 `scripts/run-prd-migrations.sh`를 작성했습니다. back1, crew, admin migration Job을 분리했고, migration 중 schema 참조 문제를 수정한 뒤 재실행해 완료했습니다. OAuth/Kakao callback URL과 S3/CloudFront asset URL도 production domain 기준으로 Secrets Manager에 반영했습니다. 외부 라우팅은 Gateway API/HTTPRoute로 구성하고, public API와 admin API를 hostname 단위로 분리했습니다.

### Result

6개 백엔드 서비스가 production namespace에서 `1/1 Running` 상태로 동작하는 것을 확인했고, `https://api.peeplist.com/healthz`, `https://admin-api.peeplist.com/healthz` health check가 정상 응답했습니다. Kakao login도 production API에서 정상 동작하는 것을 검증했습니다. API 부하 테스트에서는 `CONCURRENCY=100` 기준 38,063건 요청, 평균 응답시간 62.7ms를 확인했고, RDS read test에서는 13,560건 SELECT 요청이 실패 없이 완료되었습니다.

## 5. 백엔드 중심 기술 스택 한 줄 요약

Go backend services, PostgreSQL, AWS RDS, AWS Secrets Manager, AWS S3, AWS CloudFront, AWS ECR, Docker, Kubernetes, Kustomize, AWS EKS, Gateway API, HTTPRoute, AWS ALB, AWS ACM, Cloudflare DNS, Bash, Metrics Server, Terraform

## 6. 면접에서 1분 안에 설명할 답변

Peeple 프로젝트에서 로컬/개발 환경 중심으로 실행되던 백엔드 서비스를 AWS EKS 기반 production 환경에서 실제로 운영 가능한 상태로 전환했습니다. login, main, community, shortform, crew, admin-api 6개 서비스를 Docker image로 빌드해 ECR에 push하고, Kustomize overlay에서 image tag와 Kubernetes Deployment, Service, ServiceAccount를 관리하도록 구성했습니다.

백엔드 관점에서는 runtime dependency 정리에 집중했습니다. DB URL, OAuth/Kakao callback URL, S3/CloudFront asset URL 같은 production 설정을 AWS Secrets Manager의 `peeple-prd-env`로 관리하고, Pod에는 secret name과 region만 주입했습니다. RDS는 private subnet에 있어 로컬에서 직접 접근하지 않고 EKS 내부 Kubernetes Job으로 back1, crew, admin migration을 실행하도록 자동화했습니다.

또한 `api.peeplist.com`과 `admin-api.peeplist.com`을 분리해 HTTPS health check를 검증했고, Kakao login이 production callback으로 정상 동작하는 것도 확인했습니다. 마지막으로 API 부하 테스트에서 38,063건 요청 평균 62.7ms, RDS read test에서 13,560건 SELECT 실패 0건을 확인해 production runtime 상태를 수치로 검증했습니다.

## 7. 영어 이력서 Bullet

- Migrated backend services from local/development execution to a production-ready AWS EKS runtime environment.
- Deployed 6 backend services, including login, main, community, shortform, crew, and admin-api, using Docker, ECR, Kustomize, and Kubernetes Deployments/Services/ServiceAccounts.
- Externalized backend runtime configuration such as database URLs, OAuth/Kakao callback URLs, and S3/CloudFront asset URLs into AWS Secrets Manager instead of embedding them in code or container images.
- Automated production RDS PostgreSQL migrations through Kubernetes Jobs inside the EKS cluster, avoiding direct local access to the private database.
- Fixed a schema reference issue during migration and completed separated migration jobs for back1, crew, and admin services.
- Configured and verified API routing for `api.peeplist.com` and `admin-api.peeplist.com` using Gateway API, HTTPRoute, ALB, and HTTPS health checks.
- Resolved Docker image architecture mismatch from Apple Silicon builds by applying `DOCKER_PLATFORM=linux/amd64` for EKS amd64 nodes.
- Validated backend runtime performance with 38,063 API requests at `CONCURRENCY=100` with 62.7ms average latency and 13,560 RDS SELECT queries with zero failures.

## 8. 이 경험에서 강조하면 좋은 백엔드 키워드

- Backend Production Readiness
- Runtime Configuration
- Externalized Configuration
- Secret-based Configuration
- AWS Secrets Manager
- PostgreSQL / AWS RDS
- DB Migration Automation
- Controlled Migration
- Kubernetes Job
- OAuth Callback
- Kakao Login
- API Routing
- Health Check
- HTTPS Endpoint Verification
- S3 / CloudFront Asset Integration
- Docker Image Build
- ECR Deployment
- Kustomize Overlay
- Kubernetes Deployment
- Service Separation
- Admin API Separation
- Load Testing
- API Performance Validation
- RDS Read Test
- Production Runbook
- Backend Ownership

## 9. 과해 보일 수 있는 표현과 안전한 대체 표현

| 과해 보일 수 있는 표현 | 안전한 대체 표현 |
| --- | --- |
| 대규모 백엔드 시스템을 운영 | AWS EKS 기반 production 환경에서 6개 백엔드 서비스 배포 및 runtime 구성을 검증 |
| 완전한 MSA 아키텍처 구축 | login, main, community, shortform, crew, admin-api 서비스를 분리 배포 |
| 무중단 배포 파이프라인 구축 | Docker build, ECR push, Kustomize image tag update, Kubernetes apply 흐름을 스크립트로 자동화 |
| 완벽한 성능 최적화 | API/RDS 부하 테스트로 현재 응답시간과 리소스 사용량을 검증 |
| DB 운영 자동화 완성 | private RDS 대상 Kubernetes Job 기반 migration 실행 흐름을 구성 |
| 모든 환경 설정을 안전하게 관리 | production runtime config를 AWS Secrets Manager 기반으로 외부화 |
| 안정적인 인증 시스템 구축 | OAuth/Kakao callback URL을 production domain 기준으로 정리하고 Kakao login 동작을 검증 |
| 글로벌 CDN 연동 완료 | S3/CloudFront 기반 production asset serving 경로와 runtime URL 설정을 구성 |
| 고가용성 API Gateway 구축 | ALB/Gateway API/HTTPRoute 기반 public API와 admin API 라우팅을 구성 |
| 운영 자동화 플랫폼 구축 | 배포, migration, secret seed, 부하 테스트 스크립트를 작성해 반복 작업을 줄임 |

## 10. 백엔드 개발자 관점에서 추가로 보완하면 좋은 항목

- API contract 문서화: 서비스별 endpoint, request/response, error code, auth requirement를 OpenAPI 등으로 정리하면 backend ownership이 더 명확해집니다.
- Migration version 관리: migration 파일 버전, 적용 순서, rollback 가능 여부, 재실행 안전성을 문서화하면 production DB 변경 안정성을 높일 수 있습니다.
- Readiness/Liveness probe 분리: `/healthz` 외에 DB, Secrets Manager, downstream dependency 상태를 구분한 readiness probe를 추가하면 배포 안정성이 좋아집니다.
- Structured logging: request id, user id, route, status, latency, error code를 구조화 로그로 남기면 production 이슈 분석이 쉬워집니다.
- Error budget/SLI 정의: latency, error rate, availability 같은 backend SLI를 정하고 부하 테스트 결과와 연결하면 운영 품질을 더 설득력 있게 보여줄 수 있습니다.
- Integration test 추가: Kakao callback, DB migration, asset URL generation, 주요 API route에 대한 integration test를 구성하면 배포 전 회귀를 줄일 수 있습니다.
- Deployment rollback 절차: image tag rollback, migration 실패 시 대응, secret 변경 rollback 절차를 runbook에 추가하면 production readiness를 더 강화할 수 있습니다.
- Connection pool 튜닝: RDS connection limit, service별 pool size, timeout, retry 설정을 측정 기반으로 조정하면 DB 안정성 관점의 백엔드 역량을 더 잘 보여줄 수 있습니다.
- Asset upload/read path 검증: S3 upload, CloudFront public URL 생성, 권한/캐시 정책을 실제 API 흐름에서 검증하면 asset integration 경험이 더 완성도 있게 정리됩니다.
- Authentication flow observability: OAuth callback success/failure rate, 401/403 원인, provider error code를 로그/지표로 분리하면 인증 운영 경험을 더 강하게 보여줄 수 있습니다.
