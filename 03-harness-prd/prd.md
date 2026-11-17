# AI PRD · Juno

> Module 3 · Harness / AI PRD. The AI product requirements doc specifying all six harness surfaces, built with the **M3 · AI PRD Builder** (archetype and starting settings from the **M3 · Harness Architecture Decider**). Paste the tool's markdown over this file.

## Problem & user

Juno helps a product manager prepare 15–20 minutes before an internal stakeholder meeting. Today, the PM manually checks SharePoint, Azure DevOps (ADO), and Microsoft Teams, reconciles conflicting signals, and decides what is relevant to the stakeholder they are about to meet.

Juno produces a private, stakeholder-aware brief organized into:

- **Watch** — risks, unresolved questions, or expectation gaps
- **Highlight** — recent accomplishments worth sharing
- **Coming Next** — upcoming work relevant to that stakeholder, with commitment status preserved

V1 supports three stakeholder lenses:

- **Clinical Proponent:** implementation readiness, intended users, workflow fit, clinical evidence, adoption readiness
- **IT Leadership:** security, API/integration readiness, technical dependencies, maintainability, scalability
- **Governance:** intended use, access, required reviews, evaluation, monitoring, drift, ownership

**Archetype:** **Tool-using copilot.** A PM is waiting for the result, the task requires live source retrieval and short reconciliation, and human judgment is still required to decide what is truly important. Juno may read and draft automatically, but it does not mutate source systems or communicate on the PM's behalf.

---

## The harness

| **Surface**     | **Specification** |
| --------------- | ----------------- |
| 01 Context      | Retrieve current Beacon roadmap, release-readiness, product-decision, ADO, Teams, governance, support, and UX evidence relevant to the selected stakeholder. Dynamic delivery signals must be fresh; stale or unavailable evidence is labeled. |
| 02 Tools        | Read/search SharePoint, ADO, and Teams; retrieve source metadata; compare evidence; rank for stakeholder relevance; draft the private meeting brief. No write/send/approval tools in V1. |
| 03 Loop         | Maximum 5 tool turns, one targeted retry, and stop after two consecutive failures from the same source. Escalate unresolved or unverifiable items to the PM. |
| 04 Memory       | Session state lasts 30 minutes. PM preferences may persist 90 days. Dynamic product facts never persist as trusted memory and are re-fetched each run. |
| 05 Permissions  | read **auto** · draft **auto** · write **blocked V1** · send **blocked V1** |
| 06 Verification | Run the **Juno Brief Gate** before display: citations, status accuracy, freshness, conflict handling, stakeholder relevance, and sensitive-conclusion checks. Failed checks become visible uncertainty. |

## 01 Context · Data Requirements

- **Required sources:**
  - SharePoint: **Beacon Quarterly Roadmap**
  - SharePoint: **November Workspace Release Readiness**, especially *Access and Migration*
  - SharePoint / meeting record: **Clinical AI Governance Working Session**
  - SharePoint / decision record: **Current Product Decisions**
  - ADO: current work items including **BEC-1842 · Preference Migration** and **BEC-1975 · Model Integration Framework Pilot**
  - Teams: **Specialty Alpha Partner Team**, **AI Platform Leadership**, **Specialty Beta Clinical Leader**, and **Product Leadership**
  - current **Support Feedback** and **UX Review Notes**

- **Deliberate exclusions:**
  - patient charts, PHI, and patient-specific model outputs — not needed for PM meeting preparation;
  - unrelated Teams conversations and unrelated project material — excluded to reduce noise and cross-team leakage;
  - superseded roadmap or decision records — excluded because stale context is worse than no context.

- **SLA of truth:**
  - ADO and Teams: fetch each run; cached content may be no older than **15 minutes**;
  - SharePoint roadmap, release-readiness, governance, and decision records: use the **latest current version**;
  - Support / UX notes: use the latest available version and label evidence older than **7 days**;
  - default Teams lookback: **14 days**.

- **When a source is unreachable:**
  - retry once if the source is material;
  - if a recent cached copy is available within its freshness window, use it and label the retrieval time;
  - otherwise return a partial brief and mark affected items **NEEDS CLARIFICATION**;
  - never silently present stale evidence as current.

## 02 Tools · System Capabilities

- **The verb list:**

| Verb | Side-effect class | Purpose |
|---|---|---|
| `search_sharepoint` | read | Find relevant roadmap, release, governance, and decision material |
| `read_sharepoint_document` | read | Read a specific SharePoint document or section |
| `search_ado` | read | Find relevant ADO work items |
| `read_ado_work_item` | read | Read status, comments, dependencies, scope, and timestamps |
| `search_teams` | read | Find relevant stakeholder signals in approved Teams scopes |
| `read_teams_thread` | read | Read the source conversation and timestamp |
| `get_source_metadata` | read | Return source title, timestamp, version, and link |
| `compare_evidence` | draft | Detect conflicting dates, status, scope, or commitment language |
| `rank_for_stakeholder` | draft | Rank evidence for the selected stakeholder lens |
| `draft_meeting_brief` | draft | Create the private Watch / Highlight / Coming Next brief |

- **Deliberate omissions:**
  - no `update_ado_work_item` — Juno's V1 job is preparation, not changing shared delivery state;
  - no `edit_sharepoint` — an incorrect edit could corrupt an authoritative source;
  - no `post_teams_message` or `send_email` — Juno must not communicate on the PM's behalf;
  - no `change_permissions` — outside Juno's role and high blast radius;
  - no governance, security, or clinical approval tool — approval remains an accountable human decision;
  - no roadmap-write tool — roadmap commitments require human product judgment and alignment.

## 03 Loop · AI Costs & Latency

- **Turn ceiling:** maximum **5 tool turns** per meeting-brief request, with no more than **1 targeted retrieval retry** after reconciliation.
- **Escalation trigger:** stop and return control to the PM when:
  - a material conflict remains after the targeted retry;
  - the same source system fails twice;
  - source freshness cannot be established;
  - an authoritative source needed for a claim is unavailable;
  - verification cannot establish the status of a surfaced item.
- **Latency and cost target:** **p95 ≤ 8 seconds** end-to-end and **≤ $0.12 per brief** on the normal path.

If the loop budget is exhausted, Juno returns the verified portion of the brief plus:

> **NEEDS CLARIFICATION — Juno could not verify this item within the retrieval budget.**

## 04 Memory · Data Requirements

- **What persists, at what scope:**
  - **turn:** transient reasoning and tool outputs;
  - **session:** selected stakeholder, retrieved evidence, selected follow-ups, and selected talking points;
  - **user:** preferred brief length, preferred Watch / Highlight / Coming Next format, recurring stakeholder presets;
  - **team:** nothing in V1.

- **Expiry:**
  - session state: **30 minutes of inactivity**;
  - user presentation preferences: **90 days**;
  - explicit relevance feedback: **30 days**;
  - dynamic facts such as roadmap status, ADO status, access, approvals, launch dates, and readiness conclusions: **do not persist** as factual memory.

- **Write rules:**
  - the PM may write durable preferences through explicit action;
  - the model may propose a preference but may not save it autonomously;
  - current source-system evidence outranks model memory;
  - when the PM corrects a presentation or relevance preference, the PM's explicit correction outranks the model;
  - a PM correction may not overwrite factual product truth from authoritative sources.

## 05 Permissions · AI Risks & Mitigations

| **Side-effect class** | **Tier** |
| --------------------- | -------- |
| read                  | **auto** |
| draft                 | **auto** |
| write                 | **blocked V1** |
| send                  | **blocked V1** |

**Justification:**

- **Read = auto:** searching and reading approved internal sources does not mutate shared state. A bad retrieval wastes time but is recoverable.
- **Draft = auto:** ranking evidence and drafting a private meeting brief affects only the initiating PM and can be edited, ignored, or discarded.
- **Write = blocked V1:** changing ADO, SharePoint, roadmap status, or permissions would change shared organizational state and could cause colleagues to act on incorrect information. It is unnecessary to test Juno's core value.
- **Send = blocked V1:** Teams posts and email leave the PM's private workspace and can be mistaken for official communication. The blast radius is larger and harder to reverse.

## 06 Verification · AI Testing & Measurement

- **The check before output ships:** every brief passes the **Juno Brief Gate**.

The gate checks:

1. **Citation coverage** — every factual claim has at least one source.
   - **Bar:** 100%.
2. **Status accuracy** — roadmap and delivery claims are labeled as **Completed, Committed, In Progress, Planned — Not Yet Committed, Discovery / Exploratory, Stakeholder Expectation,** or **Unverified**.
   - **Bar:** 100% labeled; zero status-inflation errors in the golden set.
3. **Conflict handling** — material disagreement between sources is detected and preserved.
   - **Bar:** ≥90% recall on the future golden set; every detected material conflict is surfaced.
4. **Freshness** — current-state claims meet the source freshness rule or are explicitly labeled stale/unverified.
   - **Bar:** 100%.
5. **Stakeholder relevance** — every surfaced item explains why it matters to the selected stakeholder.
   - **Bar:** 100% structural coverage; target ≥80% relevance in PM testing.
6. **Sensitive conclusions** — Juno does not independently declare clinical sufficiency, governance approval, security approval, deployment safety, or adequate model performance.
   - **Bar:** zero unsupported sensitive conclusions.

- **Failure behaviour:**
  - do not silently repair or drop material uncertainty;
  - label unsupported claims **UNVERIFIED**;
  - label unresolved conflicts **NEEDS CLARIFICATION** and show the conflicting evidence;
  - do not present stale evidence as current;
  - block unsupported clinical, governance, or security conclusions and report documented status only;
  - if fewer than **2 verified items** survive the gate, return:

> **INSUFFICIENT CURRENT EVIDENCE TO PREPARE A RELIABLE BRIEF**

## Eval plan

*Stub for Module 5.* Build the human evaluation rubric in `05-evals/human-rubric.md` and define the pass bar there.

Initial scenarios to include:

- Model Integration Framework described as "self-service" despite manual steps remaining;
- broader AI Summary access expected by stakeholders but not committed;
- full workspace customization assumed at launch but not in current scope;
- a valid accomplishment that may be highlighted without inventing impact;
- a valid upcoming item that must retain a planned/not-committed label;
- a governance or monitoring question where Juno must return **NEEDS CLARIFICATION** rather than infer approval.

## Out of scope

The following are deliberately outside V1:

- writing to ADO or SharePoint;
- posting to Teams;
- sending email;
- changing permissions or access;
- changing roadmap commitments;
- approving governance, security, clinical readiness, or model performance;
- autonomous escalation;
- autonomous stakeholder outreach;
- retrieving patient-level PHI or patient-specific clinical data;
- any capability not listed in the V1 verb list.

These are product decisions, not accidental omissions.
