# DevOps 프로젝트 이력서 정리

## 핵심 메시지

AWS EKS 기반 production 인프라를 Terraform으로 설계하고, 배포/보안/라우팅/모니터링/부하 검증/비용 운영까지 구성했습니다.

## 1. DevOps 이력서 프로젝트 제목 후보

1. Peeple AWS EKS Production Infrastructure 구축
2. Terraform 기반 AWS Production Kubernetes 환경 전환
3. AWS EKS 기반 백엔드 Production 인프라 설계 및 운영 자동화
4. Kubernetes / IaC 기반 Peeple Production 배포 플랫폼 구축
5. AWS EKS, Terraform, Gateway API 기반 Production 운영 환경 구축

## 2. 한국어 이력서 Bullet

- 로컬/개발 환경 중심의 백엔드 서비스를 AWS EKS 기반 production Kubernetes 환경으로 전환하기 위해 인프라 설계, IaC, 배포, 보안, 라우팅, 모니터링 구성을 담당
- Terraform으로 production 인프라를 IaC화하고, `prd-data`와 `prd` stack을 분리하여 RDS/S3/ECR 등 데이터 리소스는 유지하면서 EKS/NAT/Node 등 비용성 compute 리소스는 필요 시 destroy/up 가능하도록 설계
- AWS EKS Kubernetes `1.34` 클러스터를 구성하고, general workload와 crew 전용 workload를 node group으로 분리하여 taint/toleration 기반 스케줄링 적용
- Kustomize 기반 `k8s/base`, `k8s/overlays/prd` 구조를 구성하여 login, main, community, shortform, crew, admin-api 서비스의 Deployment, Service, ServiceAccount, env patch, ECR image tag를 환경별로 관리
- AWS Load Balancer Controller, Gateway API, GatewayClass, Gateway, HTTPRoute를 구성하여 `api.peeplist.com`, `admin-api.peeplist.com` 도메인의 ALB 기반 HTTPS 라우팅 및 HTTP to HTTPS 301 redirect 적용
- Cloudflare DNS 환경에서 Terraform으로 ACM 인증서를 생성하고 DNS validation record를 output으로 노출하여 수동 DNS 등록과 AWS ACM 인증서 발급 흐름을 정립
- AWS Secrets Manager와 IRSA를 연동하여 production runtime secret을 `peeple-prd-env`로 통합 관리하고, Pod가 필요한 secret만 읽도록 IAM 권한 구성
- private subnet의 RDS에 직접 로컬 접근하지 않고, EKS 내부 Kubernetes Job으로 back1, crew, admin migration을 수행하는 자동화 스크립트 작성 및 migration 완료 검증
- Apple Silicon에서 빌드한 Docker image가 EKS amd64 node에서 실행되지 않는 platform mismatch 문제를 `DOCKER_PLATFORM=linux/amd64` 빌드 흐름으로 해결
- Metrics Server, RDS CloudWatch Alarm, API/RDS 부하 테스트 스크립트를 구성하고 `CONCURRENCY=100` 기준 API 38,063건, RDS read 13,560건 무실패 결과를 통해 production 리소스 상태를 검증

## 3. DevOps 포지션에 맞는 강한 버전 Bullet

- Terraform 기반으로 AWS production 인프라를 `data stack`과 `compute stack`으로 분리 설계하여, RDS/S3/ECR 데이터 리소스는 보존하고 EKS/NAT/Node 등 고비용 compute 리소스는 필요 시 destroy/up 가능한 비용 최적화 운영 구조를 구축
- AWS EKS, Kustomize, ECR, IRSA, Secrets Manager를 연계하여 6개 백엔드 서비스의 production 배포 환경을 구성하고, 서비스별 image tag, 환경 변수, IAM 권한, runtime secret을 환경별로 분리 관리
- AWS Load Balancer Controller와 Gateway API를 활용해 ALB 기반 외부 트래픽 라우팅을 구성하고, Cloudflare DNS, ACM, HTTPS listener, HTTPRoute redirect를 연계하여 production 도메인의 HTTPS 접근을 검증
- private RDS 접근 제약을 고려해 Kubernetes Job 기반 migration 자동화 흐름을 설계하고, Secrets Manager에서 DATABASE_URL을 주입받아 back1, crew, admin migration을 EKS 내부에서 수행하도록 구성
- Metrics Server, CloudWatch Alarm, API/RDS 부하 테스트 스크립트를 통해 운영 관측성을 확보하고, API 38,063건 평균 62.7ms 및 RDS SELECT 13,560건 실패 0건의 부하 검증 결과를 기반으로 리소스 사용량을 확인

## 4. STAR 형식 상세 설명

### Situation

기존 백엔드 서비스는 로컬/개발 환경 중심으로 운영되고 있었고, 실제 production 환경에서 사용할 수 있는 AWS 기반 Kubernetes 인프라, 보안 구성, 배포 흐름, DNS/HTTPS, migration, 모니터링 체계가 필요했습니다.

### Task

AWS EKS 기반 production 환경을 설계하고, Terraform IaC로 인프라를 재현 가능하게 만들며, 서비스 배포, 외부 라우팅, secret 관리, RDS migration, asset serving, 운영 지표 확인, 부하 검증까지 가능한 구조를 구축해야 했습니다.

### Action

Terraform stack을 `prd-data`와 `prd`로 분리해 RDS/S3/ECR 같은 데이터 리소스와 EKS/NAT/Node 같은 compute 리소스의 생명주기를 분리했습니다. EKS 클러스터와 node group을 구성하고, crew workload에는 taint/toleration 기반 스케줄링을 적용했습니다. Kustomize overlay로 production 배포 구성을 분리하고, ECR image tag, ServiceAccount, env patch를 관리했습니다. AWS Load Balancer Controller와 Gateway API로 ALB 라우팅을 구성하고, Cloudflare DNS와 ACM 인증서를 연결해 HTTPS 도메인을 활성화했습니다. Secrets Manager와 IRSA를 연동해 Pod 단위 secret 접근 권한을 구성했으며, private RDS migration은 Kubernetes Job 기반으로 자동화했습니다. 또한 Metrics Server, RDS CloudWatch Alarm, API/RDS 부하 테스트 스크립트를 작성해 운영 검증 체계를 마련했습니다.

### Result

`api.peeplist.com/healthz`, `admin-api.peeplist.com/healthz` HTTPS health check가 정상 응답하도록 production ingress 구성을 완료했습니다. API 부하 테스트에서 `CONCURRENCY=100` 기준 총 38,063건 요청, 평균 응답시간 62.7ms를 확인했고, RDS read test에서는 13,560건 SELECT 요청이 실패 없이 완료되었습니다. 또한 EKS/NAT/Node를 필요 시 up/down할 수 있는 구조를 통해 production 검증 환경의 compute 비용을 통제할 수 있게 했습니다.

## 5. 기술 스택 한 줄 요약

AWS EKS, RDS PostgreSQL, ECR, S3, CloudFront, ACM, ALB, Secrets Manager, IAM/IRSA, CloudWatch, Terraform, Kubernetes, Kustomize, Gateway API, AWS Load Balancer Controller, Cloudflare DNS, Docker, Bash, Metrics Server

## 6. 면접에서 1분 안에 설명할 답변

Peeple 프로젝트에서 로컬/개발 환경 중심의 백엔드 서비스를 AWS EKS 기반 production Kubernetes 환경으로 전환하는 인프라 구축을 담당했습니다. Terraform으로 `prd-data`와 `prd` stack을 분리해서 RDS, S3, ECR 같은 데이터 리소스는 유지하고, EKS, NAT Gateway, Node Group 같은 비용성 리소스는 필요할 때만 올리고 내릴 수 있도록 설계했습니다.

배포 측면에서는 Kustomize overlay로 production 환경을 분리하고, ECR image tag, ServiceAccount, env patch를 서비스별로 관리했습니다. 외부 트래픽은 AWS Load Balancer Controller와 Gateway API를 사용해 ALB 기반으로 구성했고, Cloudflare DNS와 ACM 인증서를 연결해 HTTPS 및 HTTP to HTTPS redirect까지 적용했습니다.

보안은 Secrets Manager와 IRSA를 연동해 Pod가 필요한 production secret만 읽도록 구성했고, private RDS migration은 EKS 내부 Kubernetes Job으로 자동화했습니다. 이후 Metrics Server와 CloudWatch Alarm, 부하 테스트 스크립트를 통해 API 38,063건 평균 62.7ms, RDS SELECT 13,560건 실패 0건을 확인하며 운영 가능성을 검증했습니다.

## 7. 영어 이력서 Bullet

- Designed and implemented AWS EKS-based production Kubernetes infrastructure to migrate backend services from local/development environments to a production-ready cloud environment.
- Built Terraform-based IaC with separate `prd-data` and `prd` stacks, preserving data resources such as RDS, S3, and ECR while allowing cost-heavy compute resources such as EKS, NAT Gateway, and nodes to be destroyed and recreated when needed.
- Configured Kubernetes production deployments with Kustomize overlays, managing service-specific Deployments, Services, ServiceAccounts, environment patches, and ECR image tags for 6 backend services.
- Implemented ALB-based external routing using AWS Load Balancer Controller, Gateway API, GatewayClass, Gateway, and HTTPRoute for `api.peeplist.com` and `admin-api.peeplist.com`.
- Integrated Cloudflare DNS with AWS ACM certificate validation and configured HTTPS listeners with HTTP-to-HTTPS 301 redirects.
- Secured runtime configuration with AWS Secrets Manager and IRSA, enabling pods to access only the required production secrets.
- Automated private RDS migrations through Kubernetes Jobs inside the EKS cluster, avoiding direct local access to the private database.
- Validated production resource behavior through load tests, handling 38,063 API requests at `CONCURRENCY=100` with 62.7ms average latency and 13,560 RDS SELECT queries with zero failures.

## 8. 이 경험에서 강조하면 좋은 키워드

- AWS Production Infrastructure
- EKS Production Deployment
- Terraform IaC
- Kubernetes Operations
- Kustomize Environment Overlay
- AWS Load Balancer Controller
- Gateway API
- ALB Ingress / Routing
- Cloudflare DNS
- ACM Certificate Validation
- HTTPS / TLS
- Secrets Manager
- IRSA
- IAM Least Privilege
- ECR Deployment Flow
- RDS Migration Automation
- Private Subnet RDS
- S3 / CloudFront Asset Serving
- Metrics Server
- CloudWatch Alarm
- Load Testing
- Cost-aware Infrastructure Design
- Compute Up/Down Operation
- Runbook Documentation

## 9. 과해 보일 수 있는 표현과 안전한 대체 표현

| 과해 보일 수 있는 표현 | 안전한 대체 표현 |
| --- | --- |
| 대규모 트래픽을 안정적으로 처리 | 부하 테스트를 통해 production 리소스 사용량과 응답 성능을 검증 |
| 완전한 DevOps 플랫폼 구축 | AWS EKS 기반 production 배포 및 운영 환경 구성 |
| 무중단 배포 시스템 구축 | Kustomize와 ECR image tag 기반 production 배포 흐름 구성 |
| 엔터프라이즈급 보안 아키텍처 설계 | Secrets Manager와 IRSA 기반 secret 접근 권한 구성 |
| 고가용성 인프라 구축 | AWS EKS, ALB, RDS 기반 production 운영 환경 구성 |
| 비용을 획기적으로 절감 | 비용성 compute 리소스를 필요 시 up/down할 수 있는 운영 구조 설계 |
| 완전 자동화된 CI/CD 구축 | build, push, secret seed, k8s apply, rollout restart 흐름을 스크립트로 자동화 |
| 클라우드 네이티브 전환 주도 | 로컬/개발 환경 중심 서비스를 AWS EKS 기반 production 환경으로 전환 |
| 모든 운영 지표 모니터링 구축 | Metrics Server와 RDS CloudWatch Alarm 기반 최소 운영 지표 수집 체계 구성 |
| 대규모 마이그레이션 자동화 | private RDS 대상 Kubernetes Job 기반 migration 자동화 구성 |
