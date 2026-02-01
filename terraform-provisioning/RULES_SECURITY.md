# 보안 규칙(rules)

## 비밀정보 관리

- 평문 비밀값/토큰/키를 `.tf`, `.tfvars`, `.md`에 기록 금지
- 비밀값은 `*.sops.yaml`로만 저장
- SOPS 파일은 `data "sops_file"`로만 읽기

## 권한/역할 관리

- AssumeRole 기반 접근을 기본으로 사용
- `assume_role_arn`은 Atlantis 워크플로우에 맞춰 주입
- 과도한 권한 추가 시 근거와 영향 범위 명시

## 상태/락 보안

- `backend.tf`는 S3 + DynamoDB lock을 사용
- 상태 저장소는 공개 금지 및 암호화 유지

## 변경 통제

- 모든 변경은 PR 기반으로 수행
- 의존 스택 변경 시 영향 스택 동시 검토
- 자동 적용(`apply_requirements` 없음) 환경에서는 추가 승인 절차 권장
