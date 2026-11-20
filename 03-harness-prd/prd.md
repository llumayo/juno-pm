# AI PRD · Juno

> Module 3 · Harness / AI PRD. The AI product requirements doc specifying all six harness surfaces for Juno's Decision Sync workflow.

## Problem & user

Juno helps a Beacon Product Manager keep product artifacts synchronized after an approved product decision changes.

Today, the PM must remember where a decision appears, search across artifacts, identify stale references, draft updates, and make each change manually.

Juno takes an approved decision, identifies the artifacts it affects, drafts the minimum necessary changes, lets the PM review the exact edits, then applies and verifies only the changes the PM approves.

**Archetype:** **Tool-using copilot.** A PM is actively reviewing the work. Juno may read, analyze, and draft automatically, but any change to a shared artifact requires explicit PM confirmation.

---

## The harness

| **Surface** | **Specification** |
|---|---|
| 01 Context | Retrieve the approved decision, Beacon source-of-truth rules, artifact registry, and current versions of relevant product artifacts. Re-fetch dynamic product truth rather than trusting memory. |
| 02 Tools | Read decisions and artifacts, search the product corpus, classify impact, draft exact edits, write approved changes, and verify the result. No send, delete, permission-change, or decision-approval tools. |
| 03 Loop | Maximum 5 tool turns per sync run with one targeted retry. Stop on ambiguity, stale content, failed writes, or failed verification. |
| 04 Memory | Keep the active review state and PM approvals during the sync session. Product facts and artifact contents are always re-fetched. |
| 05 Permissions | read **auto** · draft **auto** · write **confirm** · send **blocked** |
| 06 Verification | Run the **Juno Sync Gate** before a proposed edit and again before and after every approved write. |

## 01 Context · Data Requirements

Juno operates against the fictional Beacon product corpus in:

`00-context/beacon-product-context.md`

- **Required sources:**
  - Beacon Product Decision Registry
  - Beacon Artifact Registry & Source-of-Truth Map
  - current Beacon product requirements
  - current configuration records
  - current UX specifications
  - current onboarding and release materials
  - active product presentations
  - relevant ADO records when delivery status matters
  - historical artifacts when needed to determine whether content should be preserved

- **Source authority:**
  - Product Decision Registry → decision status and scope
  - approved requirements → product behavior
  - configuration records → configurable/default properties
  - ADO / release-readiness → delivery status
  - current derived artifacts → inherit approved product truth

- **Freshness:**
  - re-read the triggering decision at the start of every sync run;
  - fetch current artifact contents before drafting a change;
  - re-read an artifact immediately before writing to confirm it has not changed since PM review.

- **Deliberate exclusions:**
  - patient-level data or PHI;
  - unrelated product artifacts;
  - sources outside the PM's existing access;
  - content unrelated to the approved decision.

If a required source is unavailable or materially conflicts with an authoritative source, return **NEEDS CLARIFICATION** rather than guessing.

## 02 Tools · System Capabilities

| Verb | Side-effect class | Purpose |
|---|---|---|
| `read_decision` | read | Retrieve decision status, scope, rationale, and exclusions |
| `search_artifacts` | read | Find artifacts potentially affected by the decision |
| `read_artifact` | read | Retrieve current artifact content |
| `get_artifact_metadata` | read | Retrieve version, owner, timestamp, and current/historical status |
| `compare_decision_to_artifact` | draft | Determine whether an artifact conflicts with the approved decision |
| `classify_artifact_impact` | draft | Classify as Update, Review, No Change, or Historical — Preserve |
| `draft_artifact_patch` | draft | Produce the minimum proposed edit |
| `write_artifact` | write | Apply an exact PM-approved edit |
| `verify_artifact_write` | read | Re-read the artifact and confirm the approved change |

- **Deliberate omissions:**
  - no product-decision approval tool;
  - no delete tool;
  - no permission-change tool;
  - no Teams post or email-send tool;
  - no ADO delivery-state update;
  - no automatic modification of historical artifacts.

## 03 Loop · AI Costs & Latency

- **Turn ceiling:** maximum **5 tool turns** per synchronization run.
- **Retry budget:** maximum **1 targeted retry** when required evidence cannot be retrieved.
- **Escalation trigger:** stop and return control to the PM when:
  - the decision is not Approved;
  - decision scope is unclear;
  - authoritative sources materially conflict;
  - an artifact changes after the PM reviewed the proposed edit;
  - a write fails;
  - post-write verification fails.

A normal run is:

> **Verify decision → find and inspect artifacts → classify and draft → PM review → write approved edits → verify**

Parallel artifact reads may be grouped into a single turn where supported.

- **Initial target:** p95 **≤ 8 seconds** for the pre-review analysis and **≤ $0.12 per sync run** on the normal path.

## 04 Memory · Data Requirements

- **What persists:**
  - **turn:** retrieved evidence and intermediate comparisons;
  - **session:** decision ID, artifact impact map, proposed edits, and PM approval/rejection state;
  - **user:** lightweight review preferences such as sort order or collapsed sections;
  - **team:** nothing in V1.

- **Expiry:**
  - active sync session: **24 hours** or until completed;
  - user preferences: **90 days**;
  - dynamic product facts and artifact contents: **do not persist as trusted memory**.

- **Write rules:**
  - Juno may preserve review state;
  - PM approval or rejection outranks the model;
  - current source-system content always outranks stored memory;
  - decision status, artifact versions, and product behavior must be re-fetched before acting.

> **Principle: remember the review state; re-fetch the product truth.**

## 05 Permissions · AI Risks & Mitigations

| **Side-effect class** | **Tier** |
|---|---|
| read | **auto** |
| draft | **auto** |
| write | **confirm** |
| send | **blocked V1** |

**Justification:**

- **Read = auto:** searching approved product sources changes nothing and is recoverable.
- **Draft = auto:** impact classifications and proposed edits remain private and reviewable.
- **Write = confirm:** changing a shared artifact can affect what teammates rely on, so the PM must approve the exact proposed edit first.
- **Send = blocked:** stakeholder communication is outside the Decision Sync workflow and introduces unnecessary blast radius.

Juno may never treat general permission to "sync the decision" as approval for an unspecified edit.

## 06 Verification · AI Testing & Measurement

Every synchronization passes the **Juno Sync Gate**.

The gate checks:

1. **Decision status** — only an **Approved** decision may be synchronized.
2. **Decision fidelity** — the proposed edit stays inside the approved decision's scope.
3. **Artifact classification** — every artifact is labeled **Update, Review, No Change,** or **Historical — Preserve**.
4. **Source authority** — the proposed change is based on the correct source of truth.
5. **PM approval** — every shared-artifact write matches an exact edit approved by the PM.
6. **Freshness** — the artifact has not changed since the PM reviewed it.
7. **Post-write verification** — Juno re-reads every changed artifact and confirms that only the approved edit was applied.

- **Failure behaviour:**
  - decision is not Approved → **NOT ELIGIBLE FOR SYNC**
  - evidence or scope is ambiguous → **NEEDS CLARIFICATION**
  - artifact changed after review → **STALE REVIEW**
  - write or verification fails → **SYNC FAILED**
  - successful write and verification → **UPDATED & VERIFIED**

Juno must never report **UPDATED & VERIFIED** based only on a successful write tool call.
