# Career

Cloud · Security · DevOps · Backend · MLOps

최종 수정일: 2026년 06월 28일

## 프로필 요약

Go 백엔드와 React Native/React 기반 프론트엔드 개발을 함께 수행하며, 모바일 앱, Admin Web, 백엔드 API가 연결된 서비스 기능을 end-to-end로 개발한 경험이 있습니다. 크루 기반 커뮤니티/미션 인증 플랫폼에서 미션 생성, 인증 제출, 자동 승인/수동 검수, 하루 1회 인증 제한, 인증 게시판 권한 제어 등 주요 정책 기능을 API 계약 설계부터 프론트 상태 처리까지 연동했습니다.

또한 기존 ECS 중심 인프라를 EKS/Kubernetes 기반 구조로 전환하기 위한 인프라 아키텍처를 설계하고, Terraform과 Kustomize를 활용해 ECR, VPC, EKS, IRSA, Gateway API routing, Kubernetes manifest, migration Job, image tag 자동 반영 스크립트 등 운영 배포를 위한 IaC 기반 구성을 설계 및 구현했습니다.

## 스킬
- AWS
- Cloud
- 보안
- DevOps
- Backend
- MLOps
- Go
- Gin
- PostgreSQL
- Redis
- React Native
- Expo
- React
- TypeScript
- TanStack Query
- Terraform
- Kubernetes
- EKS
- Kustomize
- GitHub Actions
- IRSA
- Gateway API

## 학력
- 2026 졸업 / 한세사이버보안고등학교

## 인턴·대외활동

### Pabilica 초기 멤버
- 기간: 2026.02 ~ 2026.04 (3개월)

#### Peeple - 크루 기반 커뮤니티/미션 인증 플랫폼

모바일 앱, Admin Web, 백엔드 API를 포함한 크루 운영 및 미션 인증 서비스 개발에 참여했습니다.

#### 기술 스택

- React Native, Expo, TypeScript, React, TanStack Query
- Go, Gin, PostgreSQL, Redis, S3
- AWS ECS, CloudFront
- Terraform, Kubernetes, EKS, Kustomize, GitHub Actions

#### 주요 기여

- 크루 기반 커뮤니티/미션 인증 서비스의 모바일 앱, Admin Web, 백엔드 API 개발에 참여
- 회원가입 약관 동의 플로우 개선
- 서비스 이용약관, 개인정보 처리방침, 마케팅 정보 수신 동의, 위치기반 서비스 이용 동의 상세 화면 연결
- 약관 항목별 네비게이션 및 동의 상태 처리 개선
- 사진, 텍스트, 위치 기반 인증 방식별 미션 제출 UI 및 API 연동
- 미션 생성 시 승인 방식, 하루 인증 제한, 인증 게시판 공개 여부 등 정책 설정값 전송
- 미션 상세/홈 화면에서 인증 상태, 완료 횟수, 참여자 상태 동기화 처리
- `approval_mode` 기반 자동 승인/검수 대기 상태 처리
- 인증 제출 응답의 `approved`, `pending`, `rejected` 상태에 따른 UI 분기
- 자동 승인 시 검수 대기 문구를 제거하고 완료 상태를 즉시 반영
- `daily_limit_enabled`, `viewer_submitted_today`, `viewer_can_submit_proof` 응답 기반 인증 버튼 상태 처리
- 중복 인증 시 `409 DAILY_PROOF_LIMIT_EXCEEDED` 에러 처리 및 데이터 refetch 적용
- 제한 비활성화 미션에서는 같은 날 여러 번 인증 가능하도록 분기
- `board_visible`, `viewer_can_view_proof_board`, `proof_board_block_reason` 기반 인증 게시판 노출 제어
- 일반 참여자와 호스트/운영진 권한을 분리해 검수 탭/인증 게시판 접근 제어
- 미션 인증 검수 목록, 승인/반려, 검수 카운트 등 운영자용 관리 흐름 연동
- 자동 승인 미션이 검수 대기 목록에 노출되지 않도록 백엔드 정책과 프론트 UI 상태 정합성 확인
- 미션 생성, 인증 제출, 인증 상태 조회, 검수 상태 관리 API 응답 필드 정의 및 프론트 연동
- 모바일/Admin 화면에서 필요한 viewer 권한 필드 및 상태 필드 설계 요청
- 백엔드 응답 shape 변경에 맞춰 타입, 매핑, UI 상태 처리 반영
- UIDatePicker 날짜 범위 충돌로 발생한 iOS 네이티브 크래시 분석
- 최소/최대 날짜 보정 및 iOS DatePicker 처리 개선으로 크래시 방지

#### Peeple 인프라 / IaC

- 기존 ECS 중심 인프라 설계를 Kubernetes/EKS 기반 구조로 전환하기 위한 인프라 아키텍처 설계
- Terraform 기반으로 ECR, VPC, EKS, AWS Load Balancer Controller IRSA, 서비스별 IRSA 모듈 구성
- Kustomize 기반 Kubernetes manifest 구조를 설계하고 base/overlay 방식으로 stg 환경 배포 구성 분리
- login, crew, main, community, shortform, admin-api 등 Go 백엔드 서비스를 Kubernetes Deployment, Service, ServiceAccount 단위로 구성
- AWS Load Balancer Controller와 Gateway API 기반 external routing 구조 설계 및 서비스별 path routing 정의
- AWS Secrets Manager와 IRSA를 활용한 서비스별 secret 접근 권한 분리 구조 설계
- RDS migration을 app startup에서 분리하고 Kubernetes Job 기반으로 별도 실행하는 migration 전략 설계
- Terraform output을 활용해 Kubernetes values/image 설정을 자동 반영하는 스크립트 작성
- GitHub Actions 기반 배포 자동화 흐름 및 image tag 반영 구조 설계

#### 이력서용 요약

- Go 백엔드와 React Native/React 기반 프론트엔드 개발을 함께 수행하며, 크루 기반 미션 인증 플랫폼에서 모바일 앱, Admin Web, 백엔드 API를 아우르는 기능 개발에 참여했습니다.
- 미션 생성, 인증 제출, 자동 승인/수동 검수, 하루 1회 인증 제한, 인증 게시판 권한 제어 등 핵심 정책 기능을 API 계약부터 프론트 상태 처리까지 연동했습니다.
- Terraform과 Kustomize를 활용해 기존 ECS 중심 인프라를 EKS/Kubernetes 기반 구조로 전환하기 위한 ECR, VPC, EKS, IRSA, Gateway API routing, migration Job, image 자동 반영 스크립트 등 IaC 기반 운영 배포 구성을 설계 및 구현했습니다.

### 한세사이버보안고등학교 교내활동
- 기간: 2023.09 ~ 2026.01 (2년 5개월)

교내 학생들의 학교생활을 위한 도우미 한움 앱 개발 및 운영

- MSA 형태의 Python FastAPI, Django 기반 모바일 앱 개발
- 온프레미스 서버 엔지니어링
- 개발망, 운영망 구축 및 서비스 제공
- 자체 모의해킹 진행
- 서비스 제공을 위한 디자인 -> 프론트 개발 -> 백엔드 개발 -> 서빙 사이클 관리 및 진행
- 대회 및 행사 진행을 위한 실시간 서비스 제공: 투표, 토토 등
- 해킹 공격 대응 및 운영망, 개발망 보완
- 내부 사정으로 인한 서비스 종료 이후 임시 서비스 제공

## 교육

### 차세대 보안리더 양성 프로그램 BEST OF THE BEST 14기 컨설팅 트랙
- 기간: 2025.06 ~ 2025.12
- 기관: 한국정보기술연구원

경험

- MLOps 환경에서의 데이터 보호 중점 보안 솔루션 개발 및 컨설팅
- MLOps 환경 AWS IaC 개발
- 클라우드 환경 데이터 보호 솔루션 개발
- 데이터 수집/식별, 라인리지, 컴플라이언스/위협 기반 진단, 오픈소스 자동 실행 및 증적 보고서 생성, one-click setup
- MLOps 사용 기업 대상 데이터 보안 컨설팅 진행

성과

- CSPM을 수행 중인 보안 기업과의 협업을 통해 기술 도입 확인서 체결
- Enterprise 생성형 AI SaaS 기업 대상 데이터 보안 컨설팅 진행
- H-스타트업 창업경진대회 최우수상
- 인공지능학회 학술대회 논문 Accept 및 포스터 발표
- 정보보호학회 동계 학술대회 논문 Accept 및 구두 발표

### 차세대 보안리더 양성 프로그램 화이트햇 스쿨 2기
- 기간: 2024.03 ~ 2024.09
- 기관: 한국정보기술연구원

경험

- 클라우드 아키텍처 설계 및 가이드 작성
- AWS 대상 인프라 자산목록표 제작
- 기업 내 AWS 환경 보안 진단 (Prowler 도구 사용)
- 인프라 구성도 작성
- 인프라 보호 체계 수립
- 인프라 AS-IS / TO-BE 작성

성과

- 클라우드 아키텍처 설계 및 가이드 작성, 기업 컨설팅 진행
- "정량적 자체 위험성 평가 체계화 : 금융권 클라우드 위험 평가 프레임워크를 중심으로" 작성

## 자격증
- 2025.07 / 정보기기운용기능사 / 한국산업인력공단
- 2025.04 / AWS Certified Cloud Practitioner / AWS
- 2023.11 / 정보기술자격(ITQ) 아래한글 A등급 / 한국생산성본부

## 수상
- 2026 / AI Hack Camp 2026 국립중앙과학관장상 (장려상) / 국립중앙과학관 / 챱츄팀
- 2025 / 2025 지방기능경기대회 클라우드 부문 2위
- 2024 / 제10회 교내 해킹방어 콘테스트 1위 / 한세사이버보안고등학교
- 2024 / 제7회 교내 해커톤(생활부문) 1위 / 한세사이버보안고등학교
- 2024 / 2024 사이버 가디언즈 경진대회 4위 (장려상) / 코리아사이버보안연합
- 2024 / 2024 육군 사이버 보안 경진대회 4위

## 논문 및 발표
- 인공지능학회 학술대회 논문 Accept 및 포스터 발표
- A Lifecycle-Based MLOps Data Protection Framework
- 정보보호학회 동계 학술대회 논문 Accept 및 구두 발표
- MLOps 환경의 데이터 보호를 위한 프레임워크
- 제8회 금융보안원 논문공모전 참가
- "정량적 자체 위험성 평가 체계화 : 금융권 클라우드 위험 평가 프레임워크를 중심으로" 작성

## 활동
- SSR 보안 동아리 운영
- 교내 컴퓨터 동아리 창설 및 부동아리장
- 제26회 동계 해킹캠프 참가
- 한세사이버보안고등학교 전학
