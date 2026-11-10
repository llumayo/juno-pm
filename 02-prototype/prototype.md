# Prototype · Juno

> Module 2 · Prototype. The working build that tests your idea, made with your build tool of choice.

## Prototype link

https://juno-risk-radar--llumayo.replit.app/

---

## What it demonstrates

Juno demonstrates one pre-meeting workflow for a product manager: choose the stakeholder you are about to meet with, then get a short brief of the most relevant **watchouts, recent accomplishments, and upcoming items** for that audience.

The prototype tests whether stakeholder-specific prioritization can shorten the manual work of reviewing SharePoint, Azure DevOps, and Teams before a meeting and help the PM decide both what needs follow-up and what positive progress is worth sharing.

---

## Debrief

- **What worked:** The stakeholder lens made the idea much more specific. Instead of showing a generic project summary, Juno changes what it surfaces based on whether the PM is preparing for a clinical, IT, or governance conversation. The Watch / Highlight / Coming Next structure also helped balance risk preparation with good news and future-facing talking points.
- **What broke / felt like a toy:** The prototype uses static mock data, so the intelligence is simulated rather than actually derived from SharePoint, ADO, and Teams. The relevance logic is also prewritten, which means the current build does not yet prove that Juno can reliably decide what matters most from real, messy source data.
- **What I'd change next pass:** I would make the source evidence and “why this matters to this stakeholder” more explicit, then test the prototype with PMs using realistic project data. I would also reduce or re-rank the number of items if users still feel they need to scan too much before a meeting.
