# aws 인프라 관리 방법(skill)

목표: 이 저장소의 스택 구조와 의존성을 이해하고 안전하게 AWS 인프라를 관리한다.

## 핵심 원칙

- 네트워크(VPC) → 보안그룹 → 암호화(KMS) → 인증/권한(IAM) → 서비스(ecs/eks) 순서로 배포한다.
- `terraform_remote_state` 의존 스택은 선행 스택이 먼저 배포되어야 한다.

## 스택 분류

- 기반 인프라: `terraform/vpc/*`, `terraform/securitygroup/*`, `terraform/s3/*`
- 보안/권한: `terraform/kms/*`, `terraform/iam/*`, `terraform/secretsmanager/*`, `terraform/ssm/*`
- 서비스: `terraform/ecs/*`, `terraform/eks/*`, `terraform/services/*`, `terraform/codebuild/*`, `terraform/codedeploy/*`
- 운영 플랫폼: `terraform/platform/*`

## 운영 규칙

- 환경별 값은 `terraform.tfvars`에서만 변경
- 공통 변수는 `var_global.tf`, `var_route53.tf`, `variables/` 유지
- 의존성 변경 시 `remote_state` 경로와 출력 이름을 반드시 맞춘다

## 장애 예방 체크

- VPC/Subnet/SG 변경 시 의존 스택(ecs/eks/codebuild)을 함께 검토
- KMS 변경 시 ssm/secretsmanager/iam 영향을 검토
