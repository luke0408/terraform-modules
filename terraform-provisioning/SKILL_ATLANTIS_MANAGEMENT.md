# atlantis 관리법(skill)

목표: `atlantis.yaml` 기준으로 안전한 plan/apply 운영을 수행한다.

## 프로젝트 구성

- `id` 워크플로우: 대부분의 AWS 스택
- `github`: `github/*` 스택
- `datadog`: `datadog/*` 스택
- `sumologic`: `sumologic/*` 스택

## 공통 동작 규칙

- `init` 단계에서 backend role ARN 주입
- `plan` 단계에서 `assume_role_arn` 변수 주입
- `apply_requirements` 없음 → 리뷰/승인 정책은 별도 프로세스로 보완 필요

## 변경 감지 규칙

- 기본 트리거: `*.tf`, `*.tfvars`
- SOPS 사용 스택: `*.sops.yaml` 포함

## 운영 체크리스트

- 변경 경로가 `projects` 목록에 포함되는지 확인
- 워크플로우가 올바른지 확인 (`id`/`github`/`datadog`/`sumologic`)
- AssumeRole ARN이 실제 계정/권한과 일치하는지 확인
