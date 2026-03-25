# Procurement Bidding System Manifesto

## Purpose

This system exists to help organizations procure goods and services through fair, controlled, auditable bidding workflows.

It supports the lifecycle from internal demand capture through sourcing, vendor response, evaluation, award, and downstream purchasing handoff.

This is not just a bid collection tool. It is a governed decision system.

## What the system optimizes for

The system is designed to optimize for:

- fairness to vendors
- auditability for procurement, finance, and compliance
- explicit workflow control
- historical reproducibility
- tenant-safe configurability
- AI-assisted implementation without erosion of domain rules

## Core principles

### 1. Procurement is workflow-governed
Important business objects must move through explicit, reviewable states.
Requisitions, bid events, submissions, evaluations, and awards are not free-form records. They are controlled domain objects with governed transitions.

### 2. Fair competition is non-negotiable
The platform must not create unfair informational or operational advantages between vendors.
Submission deadlines, clarification handling, visibility rules, and award logic must be consistently enforced.

### 3. Auditability is more important than convenience
The system must preserve enough history to reconstruct what happened, who acted, when they acted, and what information formed the basis of a decision.
A convenient implementation that weakens traceability is not acceptable.

### 4. Business intent must be separated from business commitment
A requisition expresses internal demand.
A bid event expresses market solicitation.
A submission expresses a vendor response.
An award expresses a governed selection decision.
A purchase order expresses a downstream commitment.
These are separate concepts and must not be collapsed into one record or one workflow.

### 5. Historical decisions depend on snapshots, not live master data
Business decisions must remain reproducible even if item catalogs, specifications, vendor records, or policies later change.
Where mutable master data affects a decision, the relevant context must be snapshotted into the transactional record.

### 6. Configuration beats hardcoding
Approval policies, evaluation methods, document requirements, vendor eligibility rules, and publication modes must be configurable by tenant or organizational policy.
The system must not hardcode one procurement practice as universally correct.

### 7. Multi-tenant safety is mandatory
Tenant boundaries are foundational.
No user, workflow, background job, report, or query may cross tenant scope except through intentionally designed platform-administration behavior.

### 8. AI agents must implement the model, not improvise it
AI-assisted development is allowed only when agents follow the manifesto, architecture constraints, entity specs, and workflow specs.
When implementation and specs conflict, specs win until humans intentionally revise them.

### 9. Commercial confidentiality must be protected
Where procurement mode requires separation of technical and commercial evaluation, the platform must enforce it in backend logic, not just hide fields in the UI.

### 10. Core records are append-oriented, not disposable
Procurement is a history-bearing domain.
Core records should transition, version, supersede, or archive rather than disappear.
Deletion of transactional history is exceptional and policy-driven.

## Domain language

### Requisition
An internal request to procure goods or services.

### Bid Event
A sourcing event such as an RFQ, RFP, sealed tender, or related bidding process.

### Vendor Invitation
A controlled invitation for a vendor to participate in an event, where the event is not publicly open.

### Submission
A vendor’s response to a bid event. A submission may begin as draft, then be finalized and locked according to workflow and deadline rules.

### Evaluation
A structured review of submissions. Evaluation may include technical qualification, commercial comparison, weighted scoring, and recommendation.

### Award
A governed decision selecting one or more vendors for one or more lines or scopes.

### Purchase Order
A downstream execution artifact created after award approval, not a substitute for award decisioning.

### Clarification
A formal question-and-answer exchange connected to a bid event. Clarifications are part of the official event history.

## Invariants

The following must always remain true unless the specs are deliberately changed:

- No buyer may alter vendor-submitted bid content.
- No submission may be accepted after close unless the bid event was formally extended or reopened through governed workflow.
- Deadline enforcement must be server-side and deterministic.
- Commercial visibility must follow the event’s evaluation policy.
- Required documents and declarations must be validated before final submission.
- An award must trace back to an eligible, evaluated submission.
- Significant transitions and decision-affecting mutations must be auditable.
- Historical bid and award records must remain interpretable even if related master data changes later.
- Core transactional records must not be hard deleted in normal business workflows.
- Tenant isolation must apply to all reads, writes, background jobs, exports, and reports.

## What this system is not

This system is not:

- a simple form builder for collecting quotes
- a spreadsheet replacement without workflow control
- a generic document portal
- a chat-first vendor collaboration tool
- a purchasing module that skips sourcing governance
- a system where rules live only in UI code

## Design consequences

Because of these principles:

- workflow states are first-class
- policy checks belong in backend/domain services
- approvals are persisted states, not UI flags
- version history matters
- snapshots matter
- audit is mandatory
- tenant scope is mandatory
- evaluation confidentiality is enforced at query level
- downstream automation must not bypass business controls

## Definition of done for any behavior change

A change is not complete unless:

- relevant entity specs were updated if business meaning changed
- relevant workflow specs were updated if transitions or guards changed
- architecture constraints remain satisfied
- tenant and audit implications were reviewed
- relevant invariants were tested
- the AI or human implementer can explain which rules governed the change

## Guidance for AI-assisted development

Before implementing behavior that affects procurement meaning, workflow, visibility, approvals, deadlines, vendor rights, evaluation, or award logic, the implementer must:

1. read this manifesto
2. read `/ai/architecture-constraints.md`
3. read the relevant entity specs
4. read the relevant workflow specs
5. summarize applicable invariants and assumptions
6. only then propose or write code