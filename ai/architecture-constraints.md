# AI Architecture Constraints

These constraints are mandatory for all AI-generated and human-written code in this repository.

They are intended to prevent silent erosion of procurement controls.

## 1. Tenant safety

- Every persisted business entity must either:
  - carry `tenant_id` directly, or
  - derive tenant scope from an immutable parent relationship.
- Every read and write must be tenant-scoped.
- Cross-tenant joins, reports, lookups, or side effects are forbidden unless explicitly implemented for platform administration.
- Background jobs must preserve tenant scope explicitly.

## 2. Workflow integrity

- Workflow-governed entities may change state only through approved workflow transition services.
- Controllers, routes, UI handlers, cron jobs, import scripts, and ad hoc admin utilities must not directly mutate workflow state.
- Allowed transitions must match `/spec/workflows/*.yaml`.
- Guard validation belongs in domain/workflow services, not only in the UI.

## 3. Submission immutability

- Vendor submission payloads are vendor-owned.
- Buyer-side mutation of vendor technical or commercial payload is forbidden.
- Vendor edits are allowed only when the workflow and deadline rules allow it.
- Finalization must preserve version history or immutable revisions according to spec.
- Any corrective handling after finalization must be modeled explicitly, not performed by silent overwrite.

## 4. Snapshot discipline

- Where mutable master data influences a procurement decision, the relevant context must be snapshotted into transactional records.
- Do not rely exclusively on current master data for historical interpretation.
- Snapshot examples may include:
  - item or service description
  - scope text
  - requested quantity basis
  - unit of measure label
  - vendor legal name
  - commercial terms basis
  - evaluation policy version

## 5. Approval boundaries

- Approval-required actions must not be completed before approval finalization.
- Pending approval must be a persisted business state.
- Approval bypass by direct service call, background job, or admin shortcut is forbidden unless explicitly modeled and authorized.

## 6. Deadline enforcement

- All deadlines must use timezone-aware timestamps.
- Server time is authoritative.
- Client time must not determine final submission acceptance.
- Deadline-close behavior must be deterministic, testable, and idempotent.
- Scheduled closing must tolerate retry without duplicating side effects.

## 7. Visibility enforcement

- Technical/commercial separation must be enforced in backend authorization and query logic.
- UI hiding alone is insufficient.
- Comparison and evaluation endpoints must respect visibility rules derived from event policy and workflow stage.

## 8. Audit requirements

- Significant business mutations must produce auditable records.
- Workflow transitions must emit audit events.
- Decision-affecting changes must preserve before/after meaning or structured event metadata.
- Audit must cover at least:
  - actor
  - action
  - entity
  - timestamp
  - tenant
  - material context for reconstruction

## 9. Idempotency

The following actions must be idempotent:

- publish
- close
- finalize submission
- award finalization
- purchase order generation
- notification dispatch triggered by a transition
- scheduler-driven lock or close operations

Retries must not duplicate business outcomes.

## 10. Soft delete and retention

- Core transactional entities must not be hard deleted in normal business flows.
- Preferred strategies are:
  - status transitions
  - archival flags
  - superseded/versioned records
  - retention-policy deletion outside normal application flows
- Hard delete requires explicit policy and must not be the default path.

## 11. API/domain separation

- API DTOs are not the domain model.
- UI convenience fields must not bypass domain rules.
- Controllers should orchestrate; domain services enforce invariants.
- Persistence structures must not be shaped solely by frontend convenience.

## 12. Money handling

- Monetary values must preserve currency.
- Total calculations must be deterministic and reproducible.
- Comparison logic must explicitly define what is being ranked:
  - unit price
  - line total
  - total bid amount
  - landed cost
  - weighted score
- Avoid ambiguous "best price" logic.

## 13. Document enforcement

- Required documents and declarations must be policy-driven.
- Submission finalization must validate document requirements server-side.
- Missing required documentation must block finalization when specified by policy.

## 14. No hidden side effects

- Business-significant side effects must be explicit.
- Side effects such as notifications, locking, downstream PO creation, or status mirroring must occur through named services, transitions, or domain events.
- Silent mutation inside low-level persistence helpers is forbidden.

## 15. No raw SQL by default

- Use the primary persistence abstraction by default.
- Raw SQL is allowed only for justified performance-critical cases.
- Raw SQL must preserve tenant scope, authorization expectations, and business invariants.
- Raw SQL must not bypass audit or workflow integrity.

## 16. No client-only policy enforcement

The following must never depend solely on frontend enforcement:

- deadlines
- approval gates
- document requirements
- evaluation visibility
- tenant scope
- role permissions
- state transition guards

## 17. Entity identity and historical traceability

- Business identities such as request numbers, event numbers, submission receipts, and award references must be stable once issued.
- External references should not be reused within a tenant when policy requires uniqueness.
- Historical records must remain traceable after supersession.

## 18. Import/export behavior

- Bulk import, CSV upload, or integration code must obey the same domain invariants as UI-driven workflows.
- Export endpoints must preserve tenant and visibility constraints.
- Reporting queries must not leak hidden commercial or technical data.

## 19. Testing expectations

For any behavior change affecting domain meaning, tests must cover the relevant invariants. At minimum, include tests where applicable for:

- tenant scope
- workflow guard enforcement
- deadline behavior
- submission immutability
- approval gating
- visibility restrictions
- audit emission
- idempotent retries

## 20. Required AI implementation behavior

Before changing business behavior, the implementer must:

1. Read `/docs/manifesto.md`
2. Read `/ai/architecture-constraints.md`
3. Read relevant `/spec/entities/*.yaml`
4. Read relevant `/spec/workflows/*.yaml`
5. Summarize applicable constraints and invariants
6. State assumptions
7. Only then propose or write code

## Mandatory checks before finalizing

- No tenant-unscoped data access
- No direct workflow-state mutation outside workflow services
- No buyer mutation of vendor submissions
- No approval bypass
- No client-only deadline enforcement
- No technical/commercial visibility leak
- No destructive delete of core transactional data
- No uncited change to business behavior without corresponding spec updates