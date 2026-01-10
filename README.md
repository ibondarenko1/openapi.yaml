..............![OpenAPI 3.0](https://img.shields.io/badge/OpenAPI-3.0-brightgreen?logo=openapiinitiative)
![License](https://img.shields.io/badge/License-Proprietary-red)
> Status: Architecture & Data Model MVP (Framework-first, implementation pending)



# Compliance Automation MVP — HIPAA Readiness Framework

This repository contains the **v1 technical foundation** for a compliance automation MVP focused on **HIPAA Security Rule readiness** for small healthcare providers.

The goal of this project is to provide a **deterministic, explainable, multi-tenant platform** that:
- collects structured compliance data via a client portal,
- evaluates controls using rule-based logic (not AI scoring),
- generates audit-ready output artifacts (reports),
- supports human-in-the-loop review and immutable publishing.

This repository is intentionally **framework-first**, not UI-first.

---

## Scope (v1)

### In scope
- Single compliance framework: **HIPAA Security Rule**
- Multi-tenant architecture
- Deterministic rules engine
- Client intake (questions + evidence)
- Report package generation (PDF / XLSX)
- Immutable audit artifacts
- Minimal RBAC (client vs internal)

### Out of scope (future versions)
- Formal audit attestation
- Automated policy quality analysis
- Task tracking / remediation execution
- MSP hierarchy and delegated tenants
- Advanced AI-driven scoring

---

## Repository Structure

This repository contains the technical foundation for a compliance automation MVP.

Current structure:

/api  
- OpenAPI contract for the client portal and compliance ingestion API

/engine  
- Compliance rule engine data model  
- Control catalog, questions and rules (materialized from design backlog)

/artifacts  
- Audit-ready output templates (risk register, gap analysis, remediation roadmap, executive summary)

/framework  
- Architecture documentation and platform design

README.md  
- Project overview and architecture description


---

## Architectural Principles

- **Deterministic first**  
  All compliance scoring is rule-based and explainable.  
  AI is used only for narrative summaries (never as a source of truth).

- **Versioned & immutable**  
  Assessments, rulesets, and published report packages are versioned.
  Published artifacts are immutable.

- **Multi-tenant by design**  
  Every domain object is scoped to a tenant.
  No cross-tenant access is possible.

- **Human-in-the-loop**  
  Internal reviewers control engine execution and publishing.
  The system does not auto-publish compliance conclusions.

---

## How This Is Intended to Be Used

1. Backend team scaffolds API using `openapi.yaml`
2. Database is seeded using `/seed/*`
3. Simple client portal is built on top of the API
4. First 1–3 real clients are onboarded
5. Feedback is collected → v1.1 iteration

This repository is **sufficient to start development immediately** without additional architectural decisions.

---

## Status

- Architecture: **locked (v1)**
- Seeds: **complete**
- Templates: **complete**
- Ready for: **development handoff / MVP build**

---

## License / Usage

This repository is currently intended for **internal development and evaluation purposes**.

---

## Contact / Ownership

Product & architecture ownership:
- Founder / Architect: Ievgen Bondarenko
- 
Copyright (c) 2026 Ievgen Bondarenko

All rights reserved.

This repository and its contents are proprietary and confidential.
Unauthorized copying, modification, distribution, sublicensing,
or use of this software, in whole or in part, is strictly prohibited
without prior written permission from the copyright holder.

This repository is provided for internal development, evaluation,
and implementation purposes only.

No license, express or implied, is granted to any third party
except by explicit written agreement.


