# Security Policy

## Scope

This repository contains AI engineering skills and checklists for planning and auditing ERP systems. It does not itself run production infrastructure or process business data.

However, security defects in the guidance can still matter if they cause an AI agent or developer to recommend unsafe architecture or verification practices.

## Reporting a security issue

If you find guidance that could materially weaken authentication, authorization, tenant isolation, secrets management, cryptography, data integrity, recovery, or software-supply-chain security, please open a GitHub issue with a minimal reproducible explanation unless disclosure itself would create immediate risk.

For sensitive disclosures, contact the repository owner directly through the contact information on the GitHub profile.

## Security principles used by this project

- evidence over confidence
- default deny
- server-side authorization
- tenant isolation
- least privilege
- segregation of duties
- exact financial arithmetic
- immutable/auditable ledger history
- transaction integrity
- idempotent retries
- tested backup restore
- secure build and dependency practices
- explicit production-readiness gates

## Important limitation

Using these skills does not replace an independent security review, penetration test, qualified accounting review, domain-owner UAT, or production-readiness review for high-risk systems.

— Hazim Batwa
