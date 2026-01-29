# aws-builder

## Purpose
Define a reusable, repo-agnostic standard for building AWS IaC with Terraform.

## Scope
Applies to any Terraform module or stack that provisions AWS resources, regardless of repository or environment.

## Standards

### Directory Design (3-Tier Standard)
- Separate reusable `modules` from environment-specific `live` and composition `stacks`.
- Use `stacks` to encode standard 3-tier compositions; `live` consumes stacks or modules with environment variables.
- Keep state separated by layer: foundation, network, edge, compute, data, observability.
- Prefer tier-aware modules for subnets and security groups.

Directory layout (reference only, no files are created by this doc):
- terraform/
- terraform/README.md
- terraform/versions.tf
- terraform/modules/
- terraform/modules/_foundation/ (tags, kms, iam-baseline)
- terraform/modules/network/ (vpc, subnet-tier, nat, endpoints, routing)
- terraform/modules/edge/ (acm, route53, cloudfront, waf, alb, nlb)
- terraform/modules/compute/ (ecs-cluster, ecs-service, ecr, eks-cluster, eks-addons, nodegroup, ec2-asg, bastion, ssm)
- terraform/modules/data/ (rds, aurora, dynamodb, elasticache, opensearch, s3, efs)
- terraform/modules/security/ (sg, nacl, iam, secretsmanager, guardduty, securityhub)
- terraform/modules/observability/ (cloudwatch, logs, xray, opentelemetry)
- terraform/modules/platform/ (ci-cd-oidc, codedeploy, codepipeline)
- terraform/stacks/ (three-tier-base, three-tier-ecs, three-tier-eks, three-tier-ec2, hybrid)
- terraform/live/ (env/region/layer)

### 3-Tier Placement
- Edge/Public: IGW, public subnets, ALB/NLB, WAF, Bastion (prefer SSM).
- Web/App: ECS/EKS/EC2 workers, internal ALB, VPC endpoints.
- Data: RDS/Aurora/ElastiCache, isolated subnets, minimal egress.

### Stacks vs Modules vs Live
- Modules: single-responsibility building blocks.
- Stacks: standard 3-tier compositions that wire modules together.
- Live: environment/region instantiations, each layer has its own state and config.

### Operational Guidance
- Use layer-separated state (foundation/network/edge/compute/data/observability).
- Connect layers only through outputs or remote_state.
- Keep modules small; use stacks for standard compositions.
- Centralize environment-specific values in `live/*/env.tfvars` when used.

### Module Layout
- `main.tf`: primary resources and locals
- `variables.tf`: inputs
- `outputs.tf`: outputs
- `versions.tf`: required providers and Terraform version constraints
- `README.md`: usage, inputs, outputs, examples

### Inputs
- Prefer a single `variable "config"` object for complex modules; use flat variables for small, focused modules.
- Use `optional(...)` fields and sensible defaults to reduce required inputs.
- Keep variable names stable and descriptive (e.g., `config`, `name`, `tags`, `subnet_ids`).
- Avoid exposing provider-specific implementation details in inputs unless required.

### Outputs
- Export identifiers (ids, arns, names) rather than full resources.
- Use clear prefixes (e.g., `vpc_id`, `bucket_arn`).
- For collections, return maps or lists with stable keys.

### Resource Naming
- Use the `this` label for the primary resource (e.g., `aws_vpc.this`).
- Use deterministic naming; avoid random suffixes unless required by AWS.

### Providers
- Do not define `provider "aws"` inside reusable modules.
- Use `required_providers` with version constraints in `versions.tf`.

### Tagging
- Support a `tags` input and propagate to all taggable resources.
- Merge standard tags with resource-specific tags if needed.
- Keep tagging behavior predictable and documented.

### Documentation
- Include a usage example and a table for inputs/outputs in `README.md`.
- Document defaults, constraints, and any AWS limitations.

### Compatibility
- Maintain backward compatibility for module inputs/outputs where possible.
- If breaking changes are necessary, note them clearly in `README.md`.

## YAML-Driven Config (Optional)
If the project uses YAML-driven config, keep a top-level `config` object and avoid breaking its schema. Otherwise, keep the input surface minimal and explicit.

## Checklist for Changes
- Apply the standard directory design when creating new stacks or live layouts.
- Keep file layout consistent within the module.
- Match input style (config object vs flat vars) to module scope.
- Preserve resource naming (`this`).
- Ensure outputs are explicit and useful.
- Avoid provider blocks inside modules.
