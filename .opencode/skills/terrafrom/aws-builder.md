# aws-builder

## Purpose
Build and update AWS Terraform modules in this repository with consistent structure, inputs, and outputs.

## Scope
Applies to modules under `aws/`, including:
- vpc
- security-group
- eks
- irsa
- oidc
- role
- s3
- ec2
- ecr
- elb
- parameter-store (plain, secure)
- cloudfront

## Repository Conventions

### Module Layout
- `main.tf`: primary resources
- `variables.tf` or `variable.tf`: inputs (note the singular form appears in `ecr`, `irsa`, `oidc`)
- `outputs.tf`: outputs (optional for some modules)

### Inputs
- Prefer a single `variable "config"` object for complex modules (vpc, eks, role, irsa, ecr, security-group, parameter-store).
- Use flat variables for small, focused modules (s3, ec2).
- Use `optional(...)` fields for flexible config objects.
- Keep variable names consistent with existing modules (e.g., `config`, `bucket_name`, `subnet_id`).

### Outputs
- Export specific identifiers (ids, arns, names), not full resources.
- Use clear prefixes for resource outputs (e.g., `vpc_id`, `eks_endpoint`).
- For collections, return maps using `for` expressions.

### Resource Naming
- Primary resources use the `this` label (e.g., `aws_vpc.this`, `aws_s3_bucket.this`).

### Providers
- Do not define `provider "aws"` inside modules. Providers are defined by the caller.

### Tagging
- Use module config tags where available (`var.config.tags`).
- Some modules include standard tags like `ManagedBy = "Terraform"` and `Writer = "Cloud-Club"`. Follow existing patterns in the module you are editing.

## YAML-Driven Config
Many modules are designed for YAML-based configuration (see `README.md`). Maintain the `config` object pattern to keep YAML compatibility when adding or changing inputs.

## Minimal Skeleton (Config-Object Module)
```hcl
variable "config" {
  type = object({
    name = string
    tags = optional(map(string))
  })
}

resource "aws_example" "this" {
  name = var.config.name
  tags = var.config.tags
}

output "example_id" {
  value = aws_example.this.id
}
```

## Checklist for Changes
- Keep file layout consistent with existing module.
- Match input style (config object vs flat vars) for that module.
- Preserve resource naming (`this`).
- Ensure outputs are explicit and useful.
- Avoid provider blocks inside modules.
