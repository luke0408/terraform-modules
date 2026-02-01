# terraform-provisioning/github

GitHub/SOPS 프로바이더를 사용해 GitHub Actions 비밀값과 변수를 관리합니다.

## 프로바이더

- GitHub: `integrations/github` (`version.tf`에서 `~> 6.0`)
- SOPS: `carlpett/sops` (`version.tf`에서 `~> 0.7.2`)

## 디렉터리 의존성 맵

### `github/_module/action`

- 목적: GitHub Actions 비밀값용 재사용 모듈
- 의존: GitHub 프로바이더 (`github_actions_secret` 리소스)
- 사용처: `github/springboot-sample`

### `github/springboot-sample`

- 목적: `springboot-sample` 리포지토리의 GitHub Actions 비밀값 관리
- 의존:
  - `_module/action` (비밀값 생성)
  - SOPS 데이터 소스 (`secrets.sops.yaml` 읽기)
  - GitHub 프로바이더 (비밀값 저장)

### `github/terraform-provisioning`

- 목적: `terraform-provisioning` 리포지토리의 GitHub Actions 변수 관리
- 의존:
  - GitHub 프로바이더 (`github_actions_variable` 리소스)
  - SOPS 프로바이더가 설정되어 있으나 현재 리소스에서는 사용되지 않음
- 상태 백엔드: S3 (`backend.tf`)

### 최상위 (`github/`)

- 공통 프로바이더 설정: `provider.tf`
- 프로바이더 버전 제약: `version.tf`

## 외부 입력

- 암호화 비밀값: `github/springboot-sample/secrets.sops.yaml`

## 참고

- 이 디렉터리에는 `terraform_remote_state` 사용이 없습니다.
