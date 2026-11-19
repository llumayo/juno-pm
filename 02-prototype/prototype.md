# Prototype · Juno

> Module 2 · Prototype. The working build that tests one piece of Juno's Decision Sync workflow.

## Prototype link

https://Juno-Decision-Sync-Review-prototype.replit.app

---

## What it demonstrates

The prototype tests the **PM review checkpoint** in Juno's Decision Sync workflow.

After an approved Beacon product decision, Juno identifies the artifacts that may be affected, classifies each as **Update, Review, No Change, or Historical — Preserve**, and shows the PM the exact current and proposed content before anything is changed.

The prototype uses **DEC-021 · Rename My Workspace to My Views** as the test scenario. 

It also includes two related navigation changes that appear in current product material but are not approved:

- **Recommended → For You** — Proposed
- **Browse → Clinical Domains** — Discussion

Juno identifies these changes but does not allow them to be synchronized, demonstrating that seeing a change in a design or discussion artifact is not the same as having an approved product decision.

### Use case

**Primary: Augmentation**  
Juno does the tedious work of finding affected artifacts, comparing current and proposed content, and preparing the change set, while the PM retains judgment over what should actually change.

**Secondary: Automation**  
The artifact search, impact classification, and proposed-edit preparation are work the PM would otherwise perform manually.

### Testable assumption

> **If Juno shows the affected artifacts, why each is affected, and the exact proposed edits in one review screen, the PM can confidently decide what to approve without manually opening every source artifact first.**

The prototype intentionally stops at the approval checkpoint. Actual artifact writes and post-write verification are specified in later modules.

---

## Debrief

- **What worked:** The Current → Proposed comparison makes each recommended edit quick to understand. Separating artifacts into **Update, Review, No Change, and Historical — Preserve** also makes it clear that Juno is not performing a simple global find-and-replace. The **Not Eligible for Synchronization** section reinforces that only approved product decisions can be propagated.

- **What broke / felt like a toy:** The prototype uses hard-coded artifacts and proposed changes, so it does not yet prove that Juno can reliably discover all affected artifacts from a real product corpus. It also stops before actual writes, so execution and post-write verification are simulated rather than tested.

- **What I'd change next pass:** Allow the PM to edit a proposed change directly when an artifact is classified as Review, rather than only approving or skipping Juno's suggestion. I would also connect the review experience to real source artifacts so Juno's impact detection and proposed edits can be tested against realistic product content.
