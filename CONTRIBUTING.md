# Contributing

Contributions are welcome when they make the skills more precise, testable, evidence-driven, and useful for real ERP engineering.

## Preferred contributions

- new executable audit checks
- better failure scenarios
- stronger accounting/inventory invariants
- concurrency and idempotency test cases
- production-readiness checks
- recovery and restore verification
- authorization and tenant-isolation checks
- manufacturing, costing, quality and traceability scenarios
- references to authoritative standards or vendor documentation
- clearer machine-readable finding formats

## Contribution rules

1. Do not add a checklist item that cannot be explained in terms of a real failure mode.
2. Prefer evidence-producing verification steps over generic advice.
3. Distinguish business policy from engineering fact.
4. Do not hard-code one framework as universally correct.
5. Do not weaken the rule that unresolved Critical findings block production readiness.
6. Do not turn screenshots, framework names, test counts, or documentation claims into proof.
7. Preserve the separation between the Planning Skill and the Engineering Audit Skill.
8. For ERP-specific changes, explain the accounting, inventory, manufacturing, quality, security, or operational invariant being protected.

## Suggested change format

```text
Problem:
Failure mode:
New/changed rule:
Evidence required:
How to test it:
Severity if violated:
Reference (if applicable):
```

## Pull requests

Keep changes focused and explain why the new rule prevents a concrete class of production failure.

— Hazim Batwa
