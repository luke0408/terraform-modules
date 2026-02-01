# gitops 구조(skill)

목표: 저장소 구조를 GitOps 관점으로 해석하고 운영 흐름을 정의한다.

## GitOps 구성 요소

- 코드 저장소: `terraform-provisioning/`
- 환경 정의: `terraform/**/terraform.tfvars`
- 상태 관리: `backend.tf` (S3 + DynamoDB lock)
- 배포 자동화: `atlantis.yaml`
- 비밀 관리: SOPS (`*.sops.yaml`)

## 변경 흐름

1. 코드 변경(스택/모듈/변수)
2. PR 생성 → Atlantis autoplan
3. 리뷰 후 apply
4. 상태는 S3에 저장, 락은 DynamoDB

## 구조 규칙

- 스택은 디렉터리 단위로 독립 배포
- 공통 모듈은 `_module`/`_modules`에만 위치
- 환경값은 `terraform.tfvars`로 분리

## 운영 가이드

- 변경은 반드시 PR로 진행(직접 apply 금지)
- 스택 의존성 변경 시 관련 스택 동시 업데이트
