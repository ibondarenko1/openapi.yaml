API_SPEC_v1 — Compliance Automation MVP (HIPAA)
Version: v1.0
Date: 2026-01-08
Owner: Product/Architecture

1. Purpose
This document defines the MVP system behavior for HIPAA readiness automation:

Multi-tenant client portal intake (questionnaire + evidence)
Deterministic rules-based engine (control results, gaps, risks, remediation)
Report generation (multi-format) with optional AI narrative summary
Immutable publishing and audit trail
This spec is human-readable and complements openapi.yaml (the API contract).
2. Scope (v1)
In scope
Single framework: HIPAA (v1)
Control set v1 (target: 40 controls)
Question set v1 (control → questions mapping)
Ruleset v1 (deterministic patterns)
Report package v1 (PDF + XLSX + optional DOCX)
Roles: client_user, internal_user
Audit events (minimal)
Out of scope (future)
Auditor role (time-bound read-only)
MSP hierarchy and delegated tenant management
Evidence content analysis (policy quality scoring)
Remediation task tracking/ticketing
Advanced version branching per rule run
3. Core Objects (Domain Model)
Tenant
User
TenantMember (membership + role)
Framework / ControlsetVersion / RulesetVersion (catalog + versions)
Control
Question
Assessment
Answer
EvidenceFile
EvidenceLink
EngineRun (optional job tracking)
ControlResult
Gap
Risk (1:1 with Gap)
RemediationAction
ReportPackage (versioned)
ReportFile
AuditEvent
4. Multi-tenant & Data Isolation
Tenant scoping
Every tenant-scoped resource must have tenant_id.
Any request that targets tenant-scoped resources must verify:
caller is authenticated
caller has active membership for tenant_id
resource.tenant_id == requested tenant_id
No cross-tenant access
Forbidden by design. Return 403 on mismatch.
5. Roles & Authorization (RBAC)
Roles:

client_user: client/clinic/MSP representative within a tenant
internal_user: platform operator/consultant
RBAC principles:

client_user can intake data (answers/evidence), submit assessment, view published outputs
internal_user can run engine, generate & publish reports, manage tenant members, complete assessment
Field-level rules:

client_user never sees tenant members list or internal-only notes (if implemented)
client_user can only access report packages when status=published
Reference: RBAC Matrix v1 (included below).

6. Status Models & Immutability
6.1 Assessment status
Allowed statuses:

draft
in_progress
submitted
completed
Allowed transitions:

draft → in_progress
in_progress → submitted
submitted → in_progress (internal only; return to client for completion)
submitted → completed (internal only)
completed → (no transitions)
Immutability:

If assessment.status == completed:
answers cannot be created/updated
evidence links cannot be created
engine cannot be run (must create a new assessment)
6.2 ReportPackage status
Allowed statuses:

draft
generated
published
Allowed transitions:

draft → generated
generated → published
published → (no transitions)
Immutability:

If report_package.status == published:
package cannot be regenerated or modified
any changes require creating a new package_version
7. End-to-End Workflow (v1)
Phase A — Tenant setup
internal_user creates tenant
internal_user invites/adds client_user membership
Phase B — Assessment creation & intake
create assessment (framework=HIPAA, versions pinned)
client_user updates assessment to in_progress
client_user enters answers (batch allowed)
Phase C — Evidence
client_user requests upload URL
uploads file to storage
registers evidence record
links evidence to assessment and optionally to control/question
Phase D — Submit (Gate 1)
client_user submits assessment
server validates completeness threshold + critical questions
if pass → assessment.status=submitted
Phase E — Engine run (internal)
internal_user runs deterministic engine
system produces outputs: control_results, gaps, risks, remediation_actions
Phase F — Internal review (Gate 2)
internal_user reviews outputs for sanity
either:
proceed to reporting
return assessment to in_progress for client completion
Phase G — Report package
internal_user creates report package draft
internal_user generates report files (PDF/XLSX/DOCX)
package.status=generated
Phase H — Publish
internal_user publishes package (immutable)
optionally sets assessment.status=completed
Phase I — Client access
client_user views package metadata (published only)
downloads zip or individual files
8. Validation Rules (v1)
8.1 Answers validation
Unique (assessment_id, question_id)
Allowed values depend on question.answer_type:
yes_no: Yes/No
yes_no_partial: Yes/No/Partial
yes_no_unknown: Yes/No/Unknown
select: must be in options.choices
date: YYYY-MM-DD; cannot be future > 1 day
8.2 Evidence validation
Max size: 25 MB (v1)
Allowed types: PDF, DOCX, XLSX, PNG, JPEG
EvidenceLink allowed only if assessment.status != completed
8.3 Submit gate (Gate 1)
answered_ratio >= 0.70 of active questions
critical questions must be answered (non-null)
N/A prohibited for critical: MFA, Encryption in transit (v1)
On failure return structured error:

answered_ratio, required_ratio
missing_critical_questions[]
missing_question_count
8.4 Engine run preconditions
assessment.status == submitted (v1 strict)
pinned versions exist
at least 1 answer exists
rerun behavior: replace outputs (delete + rebuild)
8.5 Report generation preconditions
outputs exist (results/gaps/risks/remediation present)
package.status == draft
formats subset of {PDF, XLSX, DOCX}
required files must be produced:
executive_summary (PDF required)
gap_register (XLSX required)
risk_register (XLSX required)
roadmap (XLSX required)
evidence_checklist (XLSX required)
8.6 Publish preconditions
package.status == generated
required files exist
internal only
publish sets immutable state
9. RBAC Matrix (v1)
Legend: ✅ allow | ❌ deny | ⚠️ allow with constraints

Auth
login/logout/me: ✅ for both
Tenants
create tenant: internal ✅, client ❌
list/get tenant: internal ✅, client ⚠️ own only
update tenant: internal ✅, client ❌
Members
manage members: internal ✅, client ❌
Catalog
frameworks/controls/questions: ✅ for both (read-only)
Assessments
create/list/get: internal ✅, client ⚠️ own tenant only
update: internal ✅, client ⚠️ metadata + draft/in_progress only
submit: internal ✅, client ⚠️ own tenant only
complete: internal ✅, client ❌
Answers
read/upsert/batch: internal ✅, client ⚠️ until assessment.completed
Evidence
upload/register/list: ✅ for both (own tenant)
link: ✅ for both until assessment.completed
download-url: ✅ for both (audit logged)
Engine
run: internal ✅, client ❌
status/results: internal ✅, client ⚠️ read-only
Reports
create/generate/publish: internal ✅, client ❌
get/download: internal ✅, client ⚠️ published only
Audit events
internal ✅ full view
client ⚠️ filtered view
10. Audit Events (Minimum Set)
System must record the following events:

tenant_created
member_invited / member_added
assessment_created
assessment_started
assessment_submitted
engine_run_started / engine_run_completed / engine_run_failed
report_package_created
report_generation_started / report_generation_completed / report_generation_failed
report_package_published
report_package_downloaded
report_file_downloaded
evidence_uploaded
evidence_linked
evidence_downloaded
AuditEvent payload should include:

tenant_id, user_id
entity_type/entity_id
relevant metadata (file_type, format, assessment_id, etc.)
11. Report Package v1 (Content)
Required artifacts:

Executive Summary (PDF) — may include AI narrative summary
Gap Register (XLSX)
Risk Register (XLSX) — 1 gap = 1 risk
Remediation Roadmap (XLSX)
Evidence Checklist (XLSX)
AI usage rule (v1):

AI is allowed only for narrative summary and plain-language explanation
Deterministic engine remains the source of truth for scoring/status
12. Non-functional Requirements (v1)
Security: strict tenant isolation; presigned URLs; audit logging
Availability: best-effort (MVP); retries for report generation
Performance: batch answer upsert; async jobs recommended for engine/reports
Rate limits (recommended):
upload-url: 30/min per tenant
answers batch: 10/min per assessment
engine/run: 3/day per assessment
report generation: 5/day per assessment
13. Versioning Rules (v1)
Assessment pins:
controlset_version_id
ruleset_version_id
ReportPackage increments:
package_version = 1..N
Published artifacts are immutable and must remain retrievable.
14. Deliverables for Dev Pack v1
This spec must be shipped with:

openapi.yaml
seed/controls.csv
seed/questions.csv
seed/rules.json
templates/* (report templates)
