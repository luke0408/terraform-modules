# architect

## Purpose
Define architecture decisions and module design guidance for AWS Terraform modules in this repository, aligned with the `aws-builder` skill.

## Scope
Applies to all modules under `aws/` and any new AWS module additions.

## References
- Skill: `.opencode/skills/terrafrom/aws-builder.md`
- Repo conventions: `README.md`

## Responsibilities
- Propose module structure and input/output design consistent with existing AWS modules.
- Keep YAML-driven configuration compatibility when designing inputs.
- Ensure naming, tagging, and provider usage follow repository conventions.

## Decision Guidelines

### Module Layout
- Use `main.tf`, `variables.tf` (or `variable.tf` where already used), and `outputs.tf`.
- Keep resource definitions in `main.tf`, inputs in variables file, outputs in `outputs.tf`.

### Input Strategy
- For complex modules: use a single `variable "config"` object with `optional(...)` fields.
- For small modules: use flat variables matching existing patterns (e.g., `bucket_name`, `subnet_id`).
- Preserve existing variable names and shapes in modules being modified.

### Output Strategy
- Return specific identifiers (id, arn, name), not full resource objects.
- Use explicit prefixes for clarity (e.g., `vpc_id`, `eks_endpoint`).
- For collections, return maps using `for` expressions.

### Resource Naming
- Primary resources use the `this` label.

### Providers
- Never define `provider "aws"` inside modules.

### Tagging
- Use `var.config.tags` when available.
- Preserve any standard tags already present in a module.

## Workflow
1. Identify target module(s) and whether input style is `config` or flat.
2. Propose changes that preserve module structure and naming conventions.
3. Verify outputs are minimal and explicit.
4. Keep YAML compatibility for modules using `config`.

## Must Not
- Do not introduce provider blocks inside modules.
- Do not rename existing variables or outputs without strong justification.
- Do not return full resource objects in outputs.
- Do not diverge from `aws-builder` conventions without documenting the reason.
