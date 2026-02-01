# 테라폼 관리 방법(skill)

목표: 이 저장소에서 Terraform 코드 품질과 상태 관리를 일관되게 유지한다.

## 구조 규칙

- 스택별 디렉터리 구조: `terraform/<service>/<account>/<region>/`
- 공통 변수: `var_global.tf`, `var_route53.tf`, `variables/`
- 환경별 값: `terraform.tfvars`

## 상태/의존성 관리

- `backend.tf`는 환경 디렉터리 단위로 구성
- 스택 간 연결은 `terraform_remote_state`만 사용
- `remote_state` 출력 변경 시, 소비 스택을 동시에 수정

## SOPS 사용 규칙

- 비밀값은 `*.sops.yaml`에만 저장
- 코드에서 `data "sops_file"`로 읽어 변수화
- 평문 비밀값/토큰 커밋 금지

## 변경 절차

- 변경 대상 스택과 의존 스택을 먼저 파악
- `terraform.tfvars` 변경 시 Atlantis autoplan 트리거 확인
- 스택별 `provider.tf`/`version.tf`에 맞춰 provider 버전 유지
