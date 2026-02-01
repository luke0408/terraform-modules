# terraform-provisioning 프로젝트 구조 문서

이 문서는 `terraform-provisioning/`의 Terraform 프로젝트 구조를 **모듈 분리**, **실제 환경 정의**, **환경 변수 정의**, **환경 배포** 관점에서 정형화해 정리합니다.

## 1) 모듈 분리

### 공통 원칙

- 리소스/기능별로 디렉터리를 분리하고, 재사용 모듈은 `_module` 또는 `_modules` 하위에 둡니다.
- 외부 모듈은 Terraform Registry 모듈을 직접 참조합니다.

### 내부 모듈

- `terraform/iam/_modules/*`: IAM 정책/사용자/AssumeRole/SOPS 관련 모듈
- `terraform/kms/_modules/sops`: SOPS용 KMS 키/권한 구성 모듈
- `terraform/ssm/_modules/*`: SSM 파라미터 모듈
- `terraform/services/demoapp/_module/service`: 서비스 스택 전용 모듈
- `terraform/codebuild/_modules/codebuild/*`: CodeBuild 배포 및 소스 인증 모듈
- `terraform/acm/zerone-id/_modules/acm`: ACM 인증서 모듈
- `github/_module/action`: GitHub Actions 비밀값 모듈

### 외부 모듈

- ECS/ALB 스택은 `terraform-aws-modules/ecs/aws`, `terraform-aws-modules/alb/aws` 사용

### 독립 Terraform 프로젝트 분리

- `terraform/`: AWS 인프라 스택
- `github/`: GitHub 리포지토리 자동화(Secrets/Variables)
- `datadog/`: Datadog integration/monitor/dashboard
- `sumologic/`: Sumo Logic collector/partition/source
- `example/`: Terraform 문법/패턴 예제
- `scripts/`: 로컬 AssumeRole/환경 세팅 유틸

## 2) 실제 환경 정의

### 환경 단위 디렉터리 구조

- `terraform/<service>/<account>/<region>/` 혹은 `terraform/<service>/<env>/` 형태로 실제 환경을 정의합니다.
- 예시:
  - `terraform/vpc/tmcd_apnortheast2/`
  - `terraform/vpc/testd_apnortheast2/`
  - `terraform/eks/tmcd_apnortheast2/tmcdapne2-nhwy/`
  - `terraform/acm/zerone-id/ap-northeast-2/`

### 환경 상태 저장소 초기화

- `terraform/init/zerone-id/init.tf`: S3 상태 저장소 및 DynamoDB 락 테이블 생성

### 환경 간 의존성

- `terraform_remote_state`로 스택 간 의존성을 연결합니다.
- 예: `vpc -> s3`, `securitygroup -> vpc`, `ecs/demo -> vpc, iam`, `iam -> kms, secretsmanager, ecs/demo, codedeploy, eks`

## 3) 환경 변수 정의

### 변수 레이어

- **기본 변수 정의**: 각 스택의 `variables.tf`
- **공통 변수**: `var_global.tf`, `var_route53.tf`, `variables/` 디렉터리
- **환경별 변수**: `terraform.tfvars` (실제 환경 디렉터리 안에 존재)
- **로컬 계산**: `locals.tf` (필요 시)

### 비밀값/민감정보

- SOPS 파일을 통해 암호화된 값을 주입
- 예시 경로:
  - `terraform/ssm/zerone-id/ap-northeast-2/secrets.sops.yaml`
  - `terraform/codebuild/zerone-id/tmcd_apnortheast2/secrets.sops.yaml`
  - `terraform/secretsmanager/zerone-id/ap-northeast-2/demo_tmcdapne2.secrets.sops.yaml`

## 4) 환경 배포

### Atlantis 기반 자동 배포

- `atlantis.yaml`에서 각 스택을 프로젝트로 정의
- 워크플로우 분리: `id`, `github`, `datadog`, `sumologic`
- 변경 감지 파일:
  - 일반 스택: `*.tf`, `*.tfvars`
  - SOPS 사용 스택: `*.sops.yaml` 포함

### 공통 배포 옵션

- `init` 시 `backend-config`에 AssumeRole ARN 주입
- `plan` 시 `assume_role_arn` 변수로 IAM Role 주입

### 로컬 배포 유틸

- `scripts/terraform_setup.sh`: AssumeRole로 임시 자격 증명 설정

## 5) 배포 순서(권장)

의존성(remote state)을 기준으로 한 권장 순서입니다.

1. `terraform/init/zerone-id` (상태 저장소/락 테이블)
2. `terraform/s3/zerone-id` (VPC flow log 대상 등)
3. `terraform/vpc/*` (네트워크 베이스)
4. `terraform/securitygroup/*` (VPC 의존 보안그룹)
5. `terraform/kms/zerone-id/ap-northeast-2` (암호화 키)
6. `terraform/secretsmanager/zerone-id/ap-northeast-2` (KMS 의존)
7. `terraform/iam/zerone-id` (KMS/Secrets/서비스 의존)
8. `terraform/ssm/zerone-id/ap-northeast-2` (KMS 의존 파라미터)
9. `terraform/ecr/zerone-id/ap-northeast-2`
10. `terraform/ecs/demo/tmcd_apnortheast2` (VPC/IAM 의존)
11. `terraform/codedeploy/zerone-id/ap-northeast-2` (IAM/ECS 의존)
12. `terraform/codebuild/zerone-id/tmcd_apnortheast2` (VPC/SecurityGroup 의존)
13. `terraform/acm/zerone-id/ap-northeast-2`
14. `terraform/route53/zerone-id`
15. `terraform/services/demoapp/tmcd_apnortheast2` (VPC 의존)
16. `terraform/eks/tmcd_apnortheast2/tmcdapne2-nhwy` (VPC 의존)
17. `terraform/platform/jenkins/tmcd_apnortheast2` (VPC/KMS 의존)

## 6) 환경 목록(실제 환경 정의)

`terraform.tfvars`가 존재하는 디렉터리가 환경 단위입니다.

| 스택 | 환경 경로 | 비고 |
| --- | --- | --- |
| vpc | `terraform/vpc/tmcd_apnortheast2` | 제품계열 tmc (ap-northeast-2) |
| vpc | `terraform/vpc/testd_apnortheast2` | 테스트 계열 (ap-northeast-2) |
| iam | `terraform/iam/zerone-id` | 계정 공통 |
| kms | `terraform/kms/zerone-id/ap-northeast-2` | 계정/리전 공통 |
| ssm | `terraform/ssm/zerone-id/ap-northeast-2` | SOPS 연동 |
| ecr | `terraform/ecr/zerone-id/ap-northeast-2` | 리포지토리 |
| securitygroup | `terraform/securitygroup/zerone-id/tmcd_apnortheast2` | VPC 의존 |
| codebuild | `terraform/codebuild/zerone-id/tmcd_apnortheast2` | 배포 파이프라인 |
| acm | `terraform/acm/zerone-id/ap-northeast-2` | 인증서 |
| platform | `terraform/platform/jenkins/tmcd_apnortheast2` | Jenkins |
| secretsmanager | `terraform/secretsmanager/zerone-id/ap-northeast-2` | SOPS 연동 |
| ecs | `terraform/ecs/demo/tmcd_apnortheast2` | demo 서비스 |
| codedeploy | `terraform/codedeploy/zerone-id/ap-northeast-2` | ECS 배포 연동 |
| s3 | `terraform/s3/zerone-id` | 상태/로그 목적지 |
| services | `terraform/services/demoapp/tmcd_apnortheast2` | 애플리케이션 서비스 |
| eks | `terraform/eks/tmcd_apnortheast2/tmcdapne2-nhwy` | EKS 클러스터 |

## 7) Atlantis 운영 가이드(요약)

### 프로젝트 분류

- `id` 워크플로우: 대부분의 AWS 스택
- `github`: GitHub 관리 스택
- `datadog`: Datadog 스택
- `sumologic`: Sumo Logic 스택

### 공통 plan/apply 정책

- `init` 단계에서 backend role ARN 주입
- `plan` 단계에서 `assume_role_arn` 변수 주입
- `apply_requirements`는 비어 있음(수동 승인 없이 적용 가능)

### 변경 감지 범위

- `*.tf`, `*.tfvars`가 기본 트리거
- SOPS가 있는 프로젝트는 `*.sops.yaml` 포함

## 8) 프로젝트 탐색 가이드

- 전체 구조 빠르게 보기: `terraform/`, `github/`, `datadog/`, `sumologic/` 최상위 디렉터리
- 실제 환경 확인: `terraform/**/terraform.tfvars` 존재 여부로 환경 분리 확인
- 모듈 재사용 확인: `_module`, `_modules` 디렉터리 탐색
