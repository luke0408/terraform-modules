# terraform-provisioning/terraform

AWS 인프라용 Terraform 스택과 모듈입니다. 이 README는 `terraform_remote_state` 기반의 스택 간 의존성과 내부 모듈 사용 관계를 정리합니다.

## 프로바이더

- AWS: 전 스택 공통 사용
- Random: `vpc/`에서 사용
- Kubernetes: `eks/`에서 사용
- SOPS: `secretsmanager/`, `ssm/`에서 사용

## 디렉터리 의존성 맵

### `acm/`

- 내부 모듈 사용: `acm/zerone-id/_modules/acm`
- 의존: 변수로 전달되는 Route53 zone id (remote state 없음)

### `codebuild/`

- 내부 모듈 사용: `codebuild/_modules/codebuild/deployment`
- 의존:
  - `vpc` remote state (서브넷, VPC 메타데이터)
  - `securitygroup` remote state (배포용 보안그룹)

### `codedeploy/`

- 의존:
  - `iam` remote state (서비스 롤)
  - `ecs/demo` remote state (클러스터/서비스/ALB 출력)
- 선택(주석 처리): `ecs/nginx` remote state

### `ecr/`

- 독립 스택 (remote state 없음)

### `ecs/`

- 외부 모듈 사용:
  - `terraform-aws-modules/ecs/aws` (클러스터/서비스)
  - `terraform-aws-modules/alb/aws`
- `ecs/nginx/tmcd_apnortheast2`는 `vpc` remote state 의존
- `ecs/demo/tmcd_apnortheast2`는 다음에 의존:
  - `vpc` remote state
  - `iam` remote state (task role)

### `eks/`

- 내부 모듈 사용: `eks/_module`
- 의존: `vpc` remote state
- 프로바이더: `aws`, `kubernetes`

### `iam/`

- 내부 모듈 사용: `iam/_modules/*`
- 의존:
  - `kms` remote state
  - `secretsmanager` remote state
  - `ecs/demo` remote state
  - `codedeploy` remote state
  - `eks` remote state

### `kms/`

- 내부 모듈 사용: `kms/zerone-id/_modules/sops`
- 의존: `iam` remote state (SOPS role ARN)

### `platform/`

- 내부 모듈 사용: `platform/jenkins/_modules/jenkins`
- 의존:
  - `vpc` remote state
  - `kms` remote state

### `route53/`

- 독립 스택 (remote state 없음)

### `s3/`

- 독립 스택 (remote state 없음)
- `vpc`에서 소비하는 출력 제공 (flow log 목적지)

### `secretsmanager/`

- 의존: `kms` remote state
- 프로바이더: `sops`

### `securitygroup/`

- 의존: `vpc` remote state

### `services/`

- 내부 모듈 사용: `services/demoapp/_module/service`
- `services/demoapp/tmcd_apnortheast2`는 `vpc` remote state 의존

### `ssm/`

- 내부 모듈 사용: `ssm/_modules/aws_ssm_parameter`
- 의존: `kms` remote state
- 프로바이더: `sops`

### `vpc/`

- `vpc/tmcd_apnortheast2`는 `s3` remote state 의존 (flow log 목적지)
- `vpc/testd_apnortheast2`는 remote state 의존 없음

### `variables/`

- 공통 변수 정의 (의존 없음)

## Remote State 엣지 요약

- `vpc` -> `s3`
- `securitygroup` -> `vpc`
- `services/demoapp` -> `vpc`
- `ecs/nginx` -> `vpc`
- `ecs/demo` -> `vpc`, `iam`
- `eks` -> `vpc`
- `codebuild` -> `vpc`, `securitygroup`
- `platform/jenkins` -> `vpc`, `kms`
- `ssm` -> `kms`
- `secretsmanager` -> `kms`
- `kms` -> `iam`
- `iam` -> `kms`, `secretsmanager`, `ecs/demo`, `codedeploy`, `eks`
- `codedeploy` -> `iam`, `ecs/demo`
