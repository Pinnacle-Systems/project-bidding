# BUSINESS REQUIREMENT DOCUMENT

**Project Title:** Controlled Procurement Bidding & Governance System (CPBGS)

---

## EXECUTIVE SUMMARY

### 0. Objective

Provide a controlled procurement platform that ensures fair bidding, prevents information leakage, and produces decisions that can be audited and reproduced years later.

### 1. Scope

The system covers the full sourcing lifecycle:

* Requisition and approvals
* Bid event creation (RFP/RFQ/Tender)
* Vendor participation and submissions
* Technical and commercial evaluation
* Award decision and audit snapshot generation

### 2. Stakeholders

* **Requisitioners:** Raise demand
* **Buyers/Procurement:** Manage sourcing events
* **Vendors:** Submit bids
* **Evaluators:** Score submissions
* **Finance/Compliance:** Review and audit decisions

### 3. Functional Capabilities

* Create and approve requisitions
* Configure bid events with lots, deadlines, and scoring rules
* Invite vendors and manage clarifications
* Accept versioned submissions with strict deadline enforcement
* Enforce two-envelope evaluation (technical first, pricing later)
* Compute rankings and support controlled override with audit
* Generate award decisions and immutable audit snapshots
* Notifications: Deadline reminders, clarification broadcasts, submission confirmations, and award notifications

### 4. Key Forms 

* **Requisition Form** (items, budget, justification)
* **Bid Event Setup Form** (lots, criteria, weights, deadlines)
* **Vendor Submission Form** (technical + commercial inputs)
* **Evaluation Scorecard** (criteria-based scoring)
* **Award Approval Form** (selection + justification)

### 5. Reporting Requirements

* Active Bid Dashboard
* Vendor Participation Report
* Evaluation & Ranking Report
* Award Summary Report
* Full Audit Trail Report
* Snapshot Verification (historical vs current data)

### 6. Timeline (Indicative)

* Requirements & Design: 2 weeks
* Core Workflows (Requisition + Bid Event): 3 weeks
* Vendor Submission & Deadline Control: 3 weeks
* Evaluation & Award Logic: 3 weeks
* Audit, Reporting & UAT: 2 weeks
* **Total: ~12–13 weeks**

---

## 7. Role-Based Functional Overview (User Journey)

The following section describes how each user interacts with the system in a simple, form-based and step-based manner. This replaces visual diagrams to ensure clarity in PDF and print formats.

### 7.1 Overall System Entry

1. User logs into the system
2. System routes user to a role-based dashboard
3. User sees actions relevant to their role:

   * Buyer / Procurement
   * Vendor
   * Evaluator
   * Admin / Compliance

---

### 7.2 Buyer / Procurement Flow

**Stage 1: Requisition**

1. Create Requisition
2. Submit for Approval
3. Manager approves

**Stage 2: Bid Event Setup**

1. Create Bid Event
2. Define Lots, Criteria, Weights, Deadlines
3. Publish Bid Event

**Stage 3: Event Management**

1. Respond to Vendor Clarifications
2. Track Event Status

**Stage 4: Decision**

1. Review Evaluation Results
2. Execute Award OR Trigger Controlled Override
3. System generates Decision Artifact

---

### 7.3 Vendor Flow

**Stage 1: Discovery**

1. Login to Vendor Dashboard
2. View Invited / Open Bid Events
3. Open Event Details

**Stage 2: Preparation**

1. Review Lots and Requirements
2. Prepare Submission

**Stage 3: Submission**

1. Upload Technical Documents per lot
2. Enter Commercial Response per lot
3. Submit / Update Responses (creates new version per lot)
4. Finalize Submission
5. Receive Confirmation

---

### 7.4 Evaluator Flow

**Stage 1: Technical Evaluation**

1. Open Assigned Evaluations
2. Review Technical Responses
3. Enter Scores in Scorecard
4. Finalize Technical Scores

**Stage 2: Commercial Evaluation (if unlocked)**

1. Review Commercial Responses
2. Complete Evaluation
3. Submit Recommendation

---

### 7.5 Admin / Compliance Flow

**User & Access Management**

1. Create / Update Users
2. Assign Roles
3. Enable / Disable Accounts

**Policy Configuration**

1. Configure Bid Rules
2. Set Deadline Policies
3. Enable Security Controls (e.g., 2FA)
4. Review Compliance Settings

---

## PART 1: THE CONCEPTUAL LAYER 

### 1. Objective & Business Value

The CPBGS is a **Workflow-Governed Decision Engine**. It transitions procurement from a document-collection process into a **System of Truth**. The platform optimizes for:

* **Information Parity:** Ensuring no vendor has an unfair informational advantage.
* **Sealed-Bid Integrity:** Protecting pricing data until technical qualifications are met.
* **Audit Permanence:** Ensuring decisions remain reproducible years later, immune to changes in master data.

### 2. The Mental Model 

#### A. The "Lot" as the Unit of Competition

Events are subdivided into **Lots** (e.g., Lot 1: Hardware, Lot 2: Services).

* Vendors bid on specific Lots.
* The system evaluates and awards each Lot independently, allowing for **Partial Awards** where different vendors win different portions of one event.

#### B. The "Two-Envelope" Digital Vault

The system mimics a physical sealed-bid process. Every vendor submission is split into:

1. **Technical Envelope:** (Opened first) Qualifications and capabilities.
2. **Commercial Envelope:** (Locked) Pricing and financial terms.
   *The Commercial Envelope is logically vaulted and only accessible once the Technical Evaluation is finalized.*

#### C. The "Decision Snapshot" (Historical Reproducibility)

Standard systems link to live data. **This system takes a "Snapshot"**—an immutable copy of all data used at the moment of the award. This ensures the audit trail never "drifts" even if a vendor changes their name or a product is deleted from the catalog.

---

## PART 2: THE SPECIFICATION LAYER 

### 3. Canonical Domain Model

| Entity                  | Description                                                                                                            | Relationship                   |
| ----------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| **Requisition**         | Internal "Source of Intent" for the spend.                                                                             | 1:1 with Bid Event.            |
| **Bid Event**           | The parent sourcing process and rule-setter.                                                                           | 1:M with Bid Lots.             |
| **Bid Lot**             | **The Unit of Competition.** Each Lot is evaluated and awarded independently.                                          | 1:M with Submission Responses. |
| **Clarification**       | Formal Q&A record (Broadcasted to all vendors).                                                                        | M:1 with Bid Event.            |
| **Submission**          | **Event-level container** (The "Master Envelope"; holds one or more lot responses and is not independently versioned). | 1:M with Submission Responses. |
| **Submission Response** | **Lot-specific bid data** (The Technical/Price bid).                                                                   | M:1 with Bid Lot.              |
| **Evaluation**          | The scored review of a Submission Response.                                                                            | 1:1 with Submission Response.  |
| **Award**               | The selection decision for a specific Lot.                                                                             | 1:1 with Decision Artifact.    |
| **Decision Artifact**   | The **immutable snapshot package** of the award.                                                                       | 1:1 with Award.                |

---

### 4. Workflow State Machines

#### 4.1 Requisition Workflow (Internal Approval)

* **Draft → Pending:** (Action: Submit for Approval).
* **Draft → Cancelled:** (Action: Request withdrawn before submission).
* **Pending → Approved:** (Action: Manager Sign-off; Budget Validated).
* **Pending → Cancelled:** (Action: Request withdrawn before approval completes).
* **Approved → Cancelled:** (Guard: Allowed only if no Published Bid Event is linked).
* **Approved → Sourced:** (Action: Linked to a Published Bid Event).

#### 4.2 Bid Event Workflow (External Sourcing)

* **Draft → Published:** (Guard: Deadline > Now; 1+ Lot exists).

* **Draft → Cancelled:** (Action: Event withdrawn before publication).

* **Published → Closed:** (Guard: Server clock > Deadline).

* **Published → Cancelled:** (Guard: Allowed with Buyer + Compliance approval; notify all vendors).

* **Closed → Tech Eval:** (Action: Unlocks Technical Envelopes).

* **Closed / Tech Eval / Comm Eval → Cancelled:** (Guard: Allowed with Compliance approval; requires reason and full audit trail).

* **Tech Eval → Comm Eval:** (Guard: All technical scores are finalized and meet minimum qualification thresholds).

* **Comm Eval → Awarding:** (Action: Unlocks Commercial Envelopes; Calculates Ranking).

* **Awarding → Awarded:** (Action: Generates immutable Decision Artifact).

### 5. Deterministic Logic & Policy Enforcement

#### 5.1 Evaluation Formula & Tie-Breaking

To ensure objective awarding, the system calculates scores as follows:

* **Formula:** `Total Score = (Technical Score × Tech Weight) + (Commercial Score × Comm Weight)`.
* **Normalization:** `Commercial Score = (Lowest Price Received / Vendor Price) × 100`.
* **Tie-Breaker:** In the event of an identical Total Score, the **Lowest Price** wins.
* **Qualification Threshold:** Vendors must meet minimum technical score thresholds to proceed to commercial evaluation.

#### 5.2 Controlled Override

If a Buyer selects a non-winner, they **must** provide a justification and trigger a secondary "Compliance Approval." The record is flagged as "Manual Selection" in the Decision Artifact.

#### 5.3 Access Control Matrix (Visibility Governance)

| Role          | Event State                                         | View Tech Envelope | View Pricing | Edit                |
| ------------- | --------------------------------------------------- | ------------------ | ------------ | ------------------- |
| **Buyer**     | Tech Eval                                           | ✅                  | ❌            | ❌                   |
| **Evaluator** | Tech Eval                                           | ✅                  | ❌            | ✅ (Scores)          |
| **Finance**   | Comm Eval                                           | ✅                  | ✅            | ❌                   |
| **Vendor**    | Published                                           | ✅ (Own Only)       | ✅ (Own Only) | ✅ (Before Deadline) |
| **Vendor**    | Closed / Tech Eval / Comm Eval / Awarding / Awarded | ✅ (Own Only)       | ✅ (Own Only) | ❌                   |

*Note: Vendor access is restricted to their own submissions only. Vendors have no visibility into other vendor data at any stage.*

#### 5.4 Locking Matrix (Immutability)

| Entity         | State     | Locked Fields (Read-Only)                                                             |
| -------------- | --------- | ------------------------------------------------------------------------------------- |
| **Bid Event**  | Published | Weights, Criteria, Lot Definitions.                                                   |
| **Submission** | Submitted | All associated Submission Responses are locked (changes require new version per lot). |
| **Evaluation** | Finalized | Scores, Comments, and Recommendations.                                                |

---

### 6. Technical Integrity & Security

#### 6.1 Two-Envelope Security Model

The separation of data is enforced via **Logical Isolation at the Domain Service layer** (not cryptographic encryption).

* Requests for `SubmissionResponse.Price` will return a `403 Unauthorized` from the backend service if the `BidEvent.Status` is not `Comm Eval` or higher.
* **Technical data remains readable throughout all subsequent states** (Comm Eval, Awarding, Awarded) and is never re-locked once exposed.

#### 6.2 Decision Snapshot Payload (The Artifact)

Upon Award, the system must capture and freeze:

* **Vendor Identity:** Legal Name, Tax ID, and Contact at time of bid.
* **Winning Bid:** The specific price, currency, and line-item breakdown.
* **Evaluation Matrix:** Full list of evaluators, individual scores, and weights used.
* **Ranking Table:** Final position of all qualified bidders.
* **Event Context:** The technical specifications active at the time of publication.
* **Award Timestamp:** Exact system time when the award decision was finalized.
* **Approver Identity:** User(s) who approved and executed the award decision.

#### 6.3 Clarification Lifecycle

* **Moderation:** Vendor questions are private until answered.
* **Broadcast:** All answers must be broadcasted to all invited vendors.
* **Cutoff:** No clarifications are permitted within **24 hours** of the deadline.

#### 6.4 Versioned Submissions (Pre-Deadline Behavior)

* Vendors may create and update Submission Responses for one or more lots multiple times while the event is in **Published** state and before the deadline.
* Each update creates a **new version** of the Submission Response for the affected lot and marks the previous version for that lot as **Superseded**.
* At any time, there is exactly one **Active Version** per vendor per lot (the latest Submission Response version for that lot).
* Vendors do not edit submitted data in place; all changes are recorded as new versions.
* At the deadline, the **latest Active Version of each Submission Response (per lot)** is locked and becomes the version used for evaluation.
* After the deadline, all versions become **read-only** and no further submissions or updates are allowed.
* The system retains all prior versions for audit and traceability, but only the final Active Version participates in scoring and ranking.

---

### 7. System Invariants

1. **Append-Only:** No transactional records (Bids, Awards) are hard-deleted.
2. **Snapshot-Based Decisions:** Once awarded, the decision artifact is the "Source of Truth."
3. **Strict Deadlines:** Submission attempts at `Deadline + 1ms` are rejected by the server clock.
4. **No Buyer Modification:** Vendor-submitted data cannot be altered by internal users.

---

### 8. Definition of Done

A feature is complete when:

* **Workflow Guards** prevent unauthorized state transitions.
* **Locking Rules** prevent modification of finalized data.
* **Access Rules** prevent price visibility during Technical Evaluation phase.
* **Evaluation Logic** produces deterministic and reproducible outcomes.
* **Decision Artifacts** contain all required audit fields for reconstruction.
