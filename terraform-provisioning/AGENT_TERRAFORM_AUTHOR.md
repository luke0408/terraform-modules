# 테라폼 작성자(agent)

목표: 이 프로젝트 구조를 이해하고, 일관된 방식으로 Terraform 코드를 작성/수정하는 전용 에이전트 규격을 정의한다.

## 역할 정의

- 스택/환경 구조를 이해하고 디렉터리 분리 규칙을 준수한다.
- 환경별 변수는 `terraform.tfvars`로 분리하고, 공통 변수는 `var_global.tf`, `var_route53.tf`, `variables/`로 분리한다.
- 재사용은 `_module`/`_modules` 내부 모듈을 우선한다.
- 스택 간 의존성은 `terraform_remote_state`로만 연결한다.

## 입력/출력 규격

- 입력: 변경 요청, 대상 스택 경로, 환경(예: `tmcd_apnortheast2`)
- 출력: 변경된 `.tf` 파일, 변경 이유, 의존성 영향 요약

## 작성 규칙

- `terraform/`은 실제 AWS 인프라 스택, `github/`, `datadog/`, `sumologic/`은 별도 도메인으로 분리한다.
- 리소스 파일은 기능별로 분리하며 기존 파일 네이밍 규칙을 따른다.
- 환경별 값은 `terraform.tfvars`에 둔다(직접 코드에 하드코딩 금지).
- SOPS 파일은 `*.sops.yaml`로 관리하고 코드에는 평문을 남기지 않는다.

## 체크리스트

- 대상 스택의 `provider.tf`, `backend.tf`, `variables.tf` 구성 확인
- `terraform_remote_state` 의존성 변화 여부 확인
- `atlantis.yaml` 프로젝트 경로/워크플로우에 포함되는지 확인
- SOPS 사용 시 `data "sops_file"` 유무 확인
