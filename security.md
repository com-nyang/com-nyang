# Security / Cloud Security 프로젝트 이력서 정리

## 핵심 메시지

서비스를 단순히 배포하는 데서 끝내지 않고, production Kubernetes 환경에서 secret, IAM, workload identity, network isolation, TLS, DB 접근을 통제 가능한 구조로 구성했습니다.

## 1. Security Engineer 이력서 프로젝트 제목 후보

1. Peeple Production Cloud Security & Kubernetes Security 구성
2. AWS EKS 기반 Production Workload Identity 및 Secret 관리 체계 구축
3. Terraform 기반 Cloud Security Baseline 및 Kubernetes 접근 제어 구성
4. AWS EKS / IRSA / Secrets Manager 기반 Production 보안 구성
5. Production Kubernetes 환경의 IAM, Secret, Network, TLS 보안 체계 구축

## 2. 한국어 이력서 Bullet

- 로컬/개발 환경 중심 백엔드 서비스를 AWS EKS production 환경으로 전환하면서 IAM, secret management, network isolation, TLS, DB 접근 제어 중심의 cloud security baseline을 구성
- EKS Pod가 node role 권한에 의존하지 않도록 IRSA를 적용하고, login, main, community, shortform, crew, admin-api 서비스별 Kubernetes ServiceAccount와 IAM Role을 분리하여 workload identity 기반 권한 경계를 설정
- 서비스별 ServiceAccount에 IAM Role ARN annotation을 연결해 Pod가 자기 서비스의 IAM Role을 assume하도록 구성하고, node role 기반 broad permission 의존을 줄임
- AWS Secrets Manager에 production runtime secret을 `peeple-prd-env`로 통합 관리하고, Kubernetes manifest에는 secret value 대신 secret name과 AWS region만 주입하여 Git/image/ConfigMap에 민감정보가 포함될 위험을 줄임
- 서비스별 IRSA role의 Secrets Manager 권한을 `secretsmanager:GetSecretValue`, `secretsmanager:DescribeSecret` 중심으로 제한하여 runtime secret 조회에 필요한 권한만 부여
- RDS PostgreSQL을 private subnet에 배치하고 security group 기반으로 application/admin client 접근만 허용하여 public internet에서 DB endpoint로 직접 접근하지 않는 구조를 구성
- 로컬 PC에서 production RDS에 직접 접근하지 않고 EKS 내부 Kubernetes Job을 통해 migration을 수행하도록 구성하여 DB 접근 경로와 실행 위치를 cluster 내부로 통제
- AWS ACM, Cloudflare DNS validation, ALB Gateway HTTPS listener를 연계해 `api.peeplist.com`, `admin-api.peeplist.com`의 TLS 접근을 구성하고 HTTP 요청은 HTTPS로 301 redirect되도록 적용
- API와 admin API를 별도 hostname으로 분리하고 Gateway API / HTTPRoute 기반으로 backend service 라우팅을 선언적으로 관리하여 운영 및 권한 분리 기반을 마련
- RDS CloudWatch alarm과 Metrics Server를 우선 적용하고 Container Insights는 즉시 활성화하지 않아, 비용과 관측 가능성의 균형을 고려한 최소 운영 모니터링 체계를 구성

## 3. Cloud Security / DevSecOps 포지션에 맞는 강한 버전 Bullet

- EKS production 환경에서 IRSA 기반 workload identity를 구성하여 Pod가 node IAM Role에 의존하지 않도록 하고, 6개 서비스별 ServiceAccount/IAM Role을 분리해 workload 단위 권한 경계를 적용
- Secrets Manager 기반 production secret 관리 체계를 구성하여 DB URL, OAuth/Kakao callback URL, S3/CloudFront 설정을 코드, image, ConfigMap, Git에서 분리하고 runtime 조회 방식으로 전환
- RDS를 private subnet에 배치하고 security group based access control을 적용했으며, migration도 EKS 내부 Kubernetes Job으로 수행하도록 설계해 production DB의 외부 노출과 직접 접근 경로를 줄임
- Cloudflare DNS와 AWS ACM DNS validation을 연계해 TLS 인증서를 발급하고, ALB Gateway HTTPS listener 및 HTTP-to-HTTPS 301 redirect를 구성하여 API 통신의 HTTPS 사용을 강제
- IRSA AccessDenied, secret path mismatch, HTTPS listener validation, ALB TargetGroup 설정 문제를 진단/수정하며 production Kubernetes 보안 구성의 인증, 권한, secret, ingress 리스크를 운영 단계에서 해소

## 4. STAR 형식 상세 설명

### Situation

기존 백엔드 서비스는 로컬/개발 환경 중심으로 운영되고 있었고, AWS production Kubernetes 환경으로 전환하면서 secret 관리, IAM 권한 분리, private DB 접근, HTTPS/TLS, DNS validation, 운영 모니터링 같은 보안 기준이 필요했습니다.

### Task

EKS 기반 production 환경에서 Pod가 과도한 node role 권한을 사용하지 않도록 workload identity를 분리하고, runtime secret을 안전하게 관리하며, RDS를 외부에서 직접 접근할 수 없도록 network isolation을 적용해야 했습니다. 또한 public API와 admin API에 HTTPS/TLS를 적용하고, migration과 운영 절차에서 실수로 민감 리소스가 노출되거나 삭제되는 위험을 줄여야 했습니다.

### Action

서비스별 Kubernetes ServiceAccount와 IAM Role을 분리하고, IRSA annotation을 통해 각 Pod가 자기 서비스의 IAM Role을 assume하도록 구성했습니다. Secrets Manager에 production runtime secret을 `peeple-prd-env`로 관리하고, manifest에는 secret value를 넣지 않고 secret name과 region만 주입했습니다. Secrets Manager 접근 권한은 `GetSecretValue`, `DescribeSecret` 중심으로 제한했습니다.

RDS PostgreSQL은 private subnet에 배치하고 security group rule로 application/admin client 접근만 허용했습니다. DB migration은 로컬 직접 접속 대신 EKS 내부 Kubernetes Job으로 실행하도록 자동화하고, back1, crew, admin migration Job을 분리했습니다. 외부 트래픽은 AWS Load Balancer Controller와 Gateway API로 관리하고, Cloudflare DNS와 AWS ACM validation을 연계해 HTTPS listener와 HTTP-to-HTTPS redirect를 구성했습니다. 운영 관측성은 RDS CloudWatch alarm과 Metrics Server를 우선 적용해 비용과 보안 모니터링의 균형을 맞췄습니다.

### Result

Pod가 node IAM Role 대신 서비스별 IRSA Role을 사용하도록 구성해 broad permission 의존을 줄였습니다. production runtime secret은 Secrets Manager 기반으로 관리되었고, Kakao login과 production callback 설정이 정상 동작하는 것을 확인했습니다. `https://api.peeplist.com/healthz`, `https://admin-api.peeplist.com/healthz`는 200 응답을 반환했고, HTTP 요청은 HTTPS로 301 redirect되었습니다. EKS 내부 Job에서 private RDS query가 성공했으며, RDS read 부하 테스트에서 13,560건 SELECT 요청이 실패 없이 완료되었습니다.

## 5. 보안 중심 기술 스택 한 줄 요약

AWS IAM, IRSA, AWS Secrets Manager, AWS EKS, Kubernetes ServiceAccount, RDS PostgreSQL, VPC, Private Subnet, Security Group, AWS ACM, AWS ALB, AWS Load Balancer Controller, Gateway API, HTTPRoute, Cloudflare DNS, Terraform, Kustomize, CloudWatch Alarm, Bash

## 6. 면접에서 1분 안에 설명할 답변

Peeple 프로젝트에서 AWS EKS production 환경을 구성하면서 DevOps 배포 자체보다 IAM, secret, network, TLS, DB 접근 제어를 통제 가능한 구조로 만드는 데 집중했습니다. 먼저 EKS Pod가 node role 권한을 직접 사용하지 않도록 IRSA를 적용했고, login, main, community, shortform, crew, admin-api 서비스별 ServiceAccount와 IAM Role을 분리했습니다. 이를 통해 Pod가 자기 workload에 연결된 role을 assume하도록 구성해 node role 기반 broad permission 의존을 줄였습니다.

민감정보는 AWS Secrets Manager의 `peeple-prd-env`로 관리했고, Kubernetes manifest에는 secret value를 넣지 않고 secret name과 region만 주입했습니다. RDS는 private subnet에 배치하고 security group 기반으로 application/admin client 접근만 허용했으며, migration도 로컬 직접 접속 대신 EKS 내부 Kubernetes Job으로 수행하도록 했습니다.

외부 API는 Cloudflare DNS와 AWS ACM DNS validation을 연계해 인증서를 발급하고, ALB Gateway HTTPS listener와 HTTP-to-HTTPS 301 redirect를 구성했습니다. 결과적으로 production API health check는 HTTPS에서 정상 응답했고, EKS 내부 Job을 통한 private RDS query와 13,560건 RDS read test도 실패 없이 검증했습니다.

## 7. 영어 이력서 Bullet

- Built a security-focused AWS EKS production environment with IAM, workload identity, secret management, network isolation, TLS, and controlled database access.
- Implemented IRSA to prevent EKS pods from relying on the node IAM role, separating Kubernetes ServiceAccounts and IAM roles for 6 backend services.
- Managed production runtime secrets in AWS Secrets Manager and injected only the secret name and AWS region into Kubernetes manifests, avoiding secret values in Git, images, and ConfigMaps.
- Restricted Secrets Manager access to read-oriented permissions such as `secretsmanager:GetSecretValue` and `secretsmanager:DescribeSecret` through service-specific IRSA roles.
- Isolated RDS PostgreSQL in private subnets and controlled access through security group rules, preventing direct public internet access to the database endpoint.
- Automated production database migrations through Kubernetes Jobs inside the EKS cluster, reducing direct local access to the private RDS instance.
- Configured HTTPS/TLS for `api.peeplist.com` and `admin-api.peeplist.com` using AWS ACM, Cloudflare DNS validation, ALB Gateway HTTPS listeners, and HTTP-to-HTTPS 301 redirects.
- Added RDS CloudWatch alarms and Metrics Server as a cost-aware baseline for production monitoring without immediately enabling higher-cost Container Insights.

## 8. 이 경험에서 강조하면 좋은 보안 키워드

- IAM least privilege
- IRSA
- Workload Identity
- Kubernetes ServiceAccount
- Service-level IAM Role
- Secret Management
- AWS Secrets Manager
- Runtime Secret Injection
- Secret Drift Reduction
- Network Isolation
- Private Subnet
- Security Group Based Access Control
- Private RDS Access
- Controlled Migration
- Kubernetes Job
- TLS / HTTPS Enforcement
- HTTP-to-HTTPS Redirect
- DNS Validation
- AWS ACM
- Cloudflare DNS
- Gateway API Security
- ALB Routing
- Kubernetes Security
- Cloud Security Baseline
- Operational Security
- Production Readiness
- Cost-aware Security Monitoring
- CloudWatch Alarm

## 9. 과해 보일 수 있는 표현과 안전한 대체 표현

| 과해 보일 수 있는 표현 | 안전한 대체 표현 |
| --- | --- |
| 완전한 Zero Trust 아키텍처 구축 | IRSA 기반 workload identity와 서비스별 IAM Role 분리를 적용 |
| 완벽한 least privilege 구현 | Secrets Manager read 권한 중심으로 서비스별 IAM Role 권한 범위를 제한 |
| 보안 취약점을 모두 제거 | node role broad permission 의존, secret value 노출, public DB 접근 같은 주요 운영 리스크를 줄임 |
| 엔터프라이즈급 보안 체계 구축 | production Kubernetes 환경의 IAM, secret, network, TLS 보안 baseline 구성 |
| 데이터베이스를 완벽히 보호 | RDS를 private subnet에 배치하고 security group 기반 접근 제어를 적용 |
| 무결한 secret 관리 체계 구축 | production runtime secret을 AWS Secrets Manager로 관리하고 manifest/Git/image에 secret value를 넣지 않는 구조로 전환 |
| 모든 통신을 완벽히 암호화 | public API domain에 ACM 기반 HTTPS listener와 HTTP-to-HTTPS redirect 적용 |
| 강력한 보안 모니터링 구축 | RDS CloudWatch alarm과 Metrics Server 기반 최소 운영 모니터링 구성 |
| 완전 자동화된 보안 운영 | secret 주입, migration Job, DNS/ACM 운영 절차를 문서화하고 일부 스크립트로 자동화 |
| 침해 방지 아키텍처 설계 | 접근 권한, secret, DB network exposure, TLS 설정 관점의 공격면을 축소 |

## 10. 보안 담당자 관점에서 추가로 개선하면 좋을 항목

- 서비스별 단일 secret 분리: 현재 운영 편의성을 위해 `peeple-prd-env` 단일 secret을 사용했다면, 장기적으로는 서비스별 secret 또는 path 분리를 적용해 한 workload가 조회 가능한 secret 범위를 더 좁힐 수 있습니다.
- Kubernetes RBAC 명시화: ServiceAccount별 Kubernetes API 권한을 최소화하고, Role/RoleBinding을 명시적으로 관리하면 cluster 내부 권한 경계를 더 분명히 만들 수 있습니다.
- NetworkPolicy 도입: namespace/service 간 east-west traffic을 제한해, Pod 간 불필요한 통신 경로를 줄일 수 있습니다.
- Secret rotation 절차: Secrets Manager rotation 또는 수동 rotation runbook을 추가해 DB credential, OAuth secret, API key 교체 절차를 표준화할 수 있습니다.
- Audit logging 강화: EKS audit log, AWS CloudTrail, ALB access log, RDS log export를 단계적으로 활성화해 인증/인가/접근 이벤트 추적성을 높일 수 있습니다.
- Policy as Code 도입: Terraform plan, Kubernetes manifest, IAM policy를 OPA/Conftest, Checkov, tfsec 같은 도구로 검증하면 보안 설정 실수를 배포 전 줄일 수 있습니다.
- Image security scanning: ECR image scan 또는 Trivy 기반 container image scanning을 배포 흐름에 추가해 취약한 base image와 dependency를 조기에 발견할 수 있습니다.
- TLS/Domain 운영 점검 자동화: 인증서 만료, DNS record drift, HTTP redirect 상태를 주기적으로 점검하는 스크립트나 알람을 추가할 수 있습니다.
- Backup/restore 검증: RDS 백업 정책뿐 아니라 실제 restore drill을 문서화하고 검증하면 production readiness 관점의 신뢰도를 높일 수 있습니다.
- 접근 권한 리뷰: IAM Role, Secrets Manager resource policy, security group rule을 정기적으로 점검하는 checklist를 runbook에 포함하면 운영 중 권한 확장을 통제하기 쉽습니다.
