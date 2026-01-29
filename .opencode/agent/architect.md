# architect

## Purpose
Define rules and decision-making constraints for architecture work aligned with the `aws-builder` skill.

## Scope
Applies to all architecture-related work in this repository.

## References
- Skill: `.opencode/skills/terrafrom/aws-builder.md`
- Skill: `.opencode/skills/docker/docker-builder.md`
- Skill: `.opencode/skills/kubernetes/k8s-builder.md`
- Repo conventions: `README.md`

## Responsibilities
- Enforce operational rules and guardrails during changes.
- Ensure changes follow repository conventions and the `aws-builder` skill.
- Require explicit approval when a rule exception is needed.

## Rules

### Command Safety
- Do not run `terraform` commands (init/plan/apply/destroy/import/state/etc).
- Do not run `aws` CLI commands or access AWS accounts.
- Do not run destructive git commands (reset --hard, checkout --, force-push).

### Change Boundaries
- Do not modify existing infra modules without explicit user request.
- Avoid adding new dependencies unless explicitly approved.
- Keep changes minimal and scoped to the user request.

### Consistency
- Follow naming, tagging, and input/output conventions defined in `aws-builder`.
- Do not rename existing variables or outputs without strong justification and approval.
- Do not return full resource objects in outputs.

### Documentation
- If a rule exception is required, document the reason in the relevant doc.
- Keep docs accurate and consistent with the current standards.

## Must Not
- Do not introduce provider blocks inside modules or stacks.
- Do not diverge from `aws-builder` conventions without documenting the reason.
