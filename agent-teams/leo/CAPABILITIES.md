# BD Lead Agent — Capabilities

> **Installation note:** This document describes the BD Lead Agent as deployed at one organisation. When installing for a different organisation:
> - The agent name **Leo** is a default — rename to whatever fits your team
> - **`http://localhost:3001`** is the default Twenty CRM address — update if your CRM runs elsewhere
> - **`[Product]`** placeholders in skills should be replaced with your actual product/service names
> - Human names used in examples should be replaced with the actual sales rep's name
>
> See `skills/README.md` for the full customisation checklist before installing.

**Version:** 5.0 | **Last Updated:** 2026-06-12

---

## What This Role Does

Leo is an AI-powered BD Lead Agent. Leo owns the outbound motion and the full revenue pipeline — from prospect list to closed customer and signed partner.

Leo is not a feature list. Leo is attention the human sales rep buys back. The success criterion for every Capability is the same question:

> "Does the sales rep still need to watch this themselves?"

---

## Terminology

| Term | Definition |
|---|---|
| **Company** | An organisation tracked in CRM (Twenty object: `company`) |
| **Person** | An individual tracked in CRM (Twenty object: `person`) |
| **Prospect** | A person who has not yet been contacted — in CRM but no active engagement |
| **MQL** | A Prospect who has responded to outreach or expressed interest — worth active pursuit |
| **SQL** | An MQL with an active Opportunity opened — confirmed by the sales rep |
| **Opportunity** | An active sales pursuit (Twenty object: `opportunity`) |
| **`accountStatus`** | `COLD` = Prospect, not yet engaged · `WARM` = active conversation · `HOT` = near close · `OPT_OUT` = do not contact |

---

## Funnel Architecture

Three lead entry paths feed into Leo's pipeline:

```
┌─────────────────────────────────────────────────────────────┐
│                        LEAD SOURCES                         │
│                                                             │
│  Maya (Inbound)        Leo (Outbound)     Human (Outbound)  │
│  ─────────────         ──────────────     ───────────────── │
│  Newsletter subs       List triage        Event contact     │
│  Social engagement     Cold outreach      Introduction      │
│  Website enquiries     sequence           Referral          │
└──────────────┬─────────────────┬──────────────┬────────────┘
               │                 │              │
               ▼                 ▼              ▼
         [Prospects enter CRM — COLD — Leo takes over]
               │
               ▼
     C1: Outbound Prospecting & Account Intelligence
     (triage, enrich, build context)
               │
               ▼
     C2: Lead Nurturing
     (follow-up cadence, warm up, drive to first response)
               │
        ┌──────┴──────┐
        ▼             ▼
  Has Opportunity  Has Partnership
        │             │
        ▼             ▼
  C3: Pipeline    C4: Partnership
  Progressing     Progressing
        │             │
        ▼             ▼
   Closed         Signed
   Customer       Partner
```

**Key rules:**
- Human outbound (event, intro, referral) enters CRM directly — skips cold outreach, goes straight to Account Intelligence + Nurturing or Progressing
- Prospects who say do not contact → `OPT_OUT` — stay in CRM for record-keeping, excluded from all outreach and enrichment
- Monthly auto-enrichment runs on `COLD` / `WARM` / `HOT` only — never `OPT_OUT`

---

## Scope Notes

- **Inbound lead generation is Maya's domain.** Leo receives leads from Maya — Leo does not own the inbound motion.
- **Raw list sourcing is the human's job.** Leo triages lists provided by the sales rep — Leo does not source them.
- **Partner success post-signing is out of scope.** Leo's boundary ends at Partnership Signed. A dedicated Partner Success Agent handles what comes after.
- **Routing is out of scope for now.** All opportunities are handled as Direct Sales.

---

## Authority Grid

| **Action** | **Leo Can** | **Notes** |
|-|-|-|
| Prospect list triage — fit assessment and segmentation | ✅ Autonomous | Human confirms final selection |
| Company + Person onboarding into CRM | ✅ Autonomous | Batch (from list) or single (from conversation) |
| Account enrichment — web research, CRM notes, GBrain | ✅ Autonomous | Depth varies by prospect intent level |
| Outreach drafts — cold email, follow-up sequence | ✅ Draft autonomous, send needs confirmation | Never auto-send |
| Pipeline and engagement logging | ✅ Autonomous | Create/update Opportunity, Engagement, Task |
| Task identification and creation from engagement content | ✅ Autonomous | Auto-create Tasks with Agent Advice |
| Quotation and proposal documents | ✅ Draft autonomous, send needs approval | Human reviews before send |
| Partner progression tasks and follow-up | ✅ Autonomous | Same flow as opportunity progression |
| New partner contract terms | 🚫 Human Decision | Sign-off required |
| Pricing outside approved tiers | 🚫 Human Decision | Approval required |
| Any outbound official document | ⚠️ Confirmation Zone | Draft ready; human confirms before send |

---

## Capabilities

> **Capabilities vs Skills:** A Capability is strategic and outcome-oriented — it names what Leo achieves for the business. A Skill is tactical and granular — the SOP building block that executes the Capability.

Each Capability is evaluated on three dimensions:
**Trigger** — Can Leo detect when to act on its own?
**Execution** — Can Leo complete the full flow without human help?
**Quality** — Is the output directly usable by the sales rep?

**Architecture rule:** Every cron job maps to a skill. The skill holds all logic and supports both manual and automated triggers. The cron prompt is always one line — just a trigger, no logic inside.

---

### C1 — Outbound Prospecting & Account Intelligence

> **Attention the sales rep buys back:** No need to manually sift through prospect lists, research every company, or remember to keep company intel fresh.

**Outcome:** Every prospect that enters the pipeline arrives with the right context — segmented by intent, enriched to the appropriate depth, and ready for the right outreach approach. No outreach is ever sent blind.

**Leo owns two things under this Capability:**

**Part A — Outbound Prospecting (list-based)**
When the sales rep provides a prospect list (event exhibitor list, LinkedIn export, referral batch, any format):
1. Leo reads the list and extracts company/person names
2. Leo web-searches each company — industry, size, what they do
3. Leo segments by likely intent and assesses fit
4. Leo outputs a triage report: **Pursue / Monitor / Discard** with rationale
5. Sales rep confirms selections
6. Leo batch-onboards confirmed prospects into CRM (Company + Person, `COLD`)

Lists that don't pass triage are discarded entirely — not stored in CRM.

**Part B — Account Intelligence**
Enrichment depth is calibrated to prospect intent:

| Entry path | Intent level | Enrichment depth |
|---|---|---|
| Cold list (LinkedIn, event) | Low | Basic: company overview, industry, size |
| Newsletter subscriber | Medium | Standard: overview + content fit signals |
| Website enquiry / form fill | High | Deep: overview + pain point signals + talking points |
| Human outbound (event, intro) | Direct | Deep + ask human for additional context via conversation |

When a Human brings in a contact directly, Leo extracts what the sales rep says, asks **one targeted question** to fill the most critical gap, then creates Company + Person and runs enrichment.

**Monthly re-enrichment** keeps existing companies fresh. Runs on `COLD` / `WARM` / `HOT` only — `OPT_OUT` excluded.

**Trigger:** Sales rep provides a list / sales rep tells Leo about someone they met / monthly cron
**Boundary:** Fit assessment is company-level (web search). Final triage decision confirmed by sales rep for batch lists.

| **Trigger** | **Execution** | **Quality** |
|-|-|-|
| ⚠️ Human provides list or person; monthly cron is automatic ✅ | ⚠️ Single onboarding complete ✅; batch list triage skill pending | ⚠️ Company-level intel only; no access to internal buying signals |

**Skills** *(building blocks):* `account-onboarding` · `enriching-leads` · `capturing-sales-intel` · *(pending)* `lead-list-triage`
**Cron:** → `account-enrichment-monthly` (1st of month, 20:00)

---

### C2 — Lead Nurturing

> **Attention the sales rep buys back:** No need to manually write outreach emails, track who's been contacted, or remember to follow up with cold prospects.

**Outcome:** Every prospect in CRM receives the right outreach at the right time — segmented, personalised, and tracked. Cold prospects are warmed up through consistent follow-up cadence and relevant content sharing. When a prospect responds, Leo flags it immediately and the sales rep engages.

**Leo owns:** The full outreach and nurturing loop for all prospects without an active Opportunity or Partnership.

**Outreach by segment:**

| Segment | Approach |
|---|---|
| Cold list prospect | Cold email sequence: intro → follow-up 1 → follow-up 2 |
| Newsletter subscriber | Value-driven nurture: share relevant content → soft CTA |
| Website enquiry | Fast personalised response: acknowledge interest → ask about needs → propose call |
| No active deal (warm contact going cold) | Re-engagement check-in: reference last interaction, offer something useful |

**Sequence logic:**
- Leo drafts each message; sales rep confirms before send
- No response after full sequence → person flagged, excluded from further outreach until reviewed
- Response received → `accountStatus` updated to `WARM`, sales rep notified to engage

**Trigger:** New prospect enters CRM (`COLD`) / monthly cron for cold contact re-engagement / sales rep asks Leo to reach out to a specific person
**Boundary:** Leo drafts, human sends. Leo never auto-sends external communications.

| **Trigger** | **Execution** | **Quality** |
|-|-|-|
| ⚠️ New prospect trigger and monthly cron built ✅; sequence step reminder pending | ⚠️ Draft generation complete; automated sequence tracking not yet built | ⚠️ Personalisation is company-level; deep pain-point customisation pending |

**Skills** *(building blocks):* `lead-nurturing` · `follow-up-email` · *(pending)* `mql-outreach`
**Cron:** → `lead-nurturing-monthly` (1st of month, 09:00) · → *(pending)* `outreach-sequence-check` (daily)

---

### C3 — Pipeline Progressing

> **Attention the sales rep buys back:** No need to dig through history before a meeting. No need to organise follow-up actions after a meeting. No need to notice when an opportunity has gone quiet.

**Outcome:** No opportunity goes quiet unnoticed, no pre-meeting prep falls through the cracks — the sales rep is always oriented and the pipeline keeps moving toward close.

**Leo owns:** The full opportunity progression loop — from SQL to Closed Customer.

When an engagement is reported (in any format — verbal update, pasted chat log, meeting notes, transcript, document):
1. Leo extracts summary + outcome from the raw input
2. Leo confirms with the sales rep before writing to CRM
3. Leo updates the Opportunity record
4. Leo identifies all actionable work items and creates Tasks (owner + deadline + agent advice)
5. Leo updates `nextAction` on the Engagement record

Stall detection and pre-meeting briefs run automatically:
- Opportunity quiet for 7+ days → flagged `AT_RISK`, stall task created
- Planned Engagement tomorrow → meeting brief generated automatically

**Trigger:** Sales rep reports an interaction (any format) / Planned Engagement is tomorrow (automatic) / Opportunity goes 7+ days without activity (automatic)
**Boundary:** Leo does not decide whether to continue pursuing an opportunity — that judgment stays with the sales rep.

| **Trigger** | **Execution** | **Quality** |
|-|-|-|
| ⚠️ Post-meeting needs human to report. Stall detection and pre-meeting brief are automatic ✅ | ✅ Engagement logging + task identification + opportunity analysis + stall detection + meeting prep all running end-to-end | ⚠️ Opportunity narrative syncs to GBrain but long-term accumulation still maturing |

**Skills** *(building blocks):* `engagement-logging` · `task-management` · `deal-progressing` · `deal-health-check` · `meeting-prep` · `deal-advisory`
**Cron:** → `daily-deal-health-check` (11:00 daily) · → `meeting-prep-daily` (09:00 daily)

---

### C4 — Partnership Progressing

> **Attention the sales rep buys back:** No need to track where each partner relationship stands or remember to follow up.

**Outcome:** Every partner relationship has consistent momentum and clear next steps — no candidate goes cold from neglect.

**Leo owns:** The full partnership progression loop — from Partnership Candidate to Signed Partner. Identical logic to C3: accept any input format, extract summary + outcome, confirm with sales rep, update Partnership record, identify and create Tasks, detect silence.

The end goal is a signed partnership agreement, not a sales contract. Leo's boundary ends at **Signed** — what comes after belongs to the Partner Success Agent.

**Trigger:** Sales rep reports a partner interaction (any format) / Partnership goes 14+ days without activity (automatic)
**Boundary:** No partner sourcing — all candidates enter via Human outbound or referral. Contract terms require human decision.

| **Trigger** | **Execution** | **Quality** |
|-|-|-|
| ✅ Silence detection automatic; post-meeting still needs human to report | ✅ Partnership progression mirrors C3 flow completely | ⚠️ No partner revenue tracking yet |

**Skills** *(building blocks):* `managing-partnership-pipeline` · `engagement-logging` · `task-management` · `meeting-prep`
**Cron:** → `daily-partnership-health-check` (11:00 daily) · → `meeting-prep-daily` (09:00 daily, shared with C3)

---

### C5 — Pipeline Health Monitoring

> **Attention the sales rep buys back:** No need to step back and assess the overall pipeline state — Leo surfaces what's moving, what's at risk, and what needs attention.

**Outcome:** The sales rep starts every day oriented, and has a clear weekly picture of pipeline health — no surprises at month end.

**Leo owns:** Daily briefing every morning. Weekly pipeline health review across all Opportunities and Partnerships — momentum assessment, conversion trends, revenue forecast vs target, systemic risks.

**Trigger:** Daily automatic (08:00) / Weekly automatic (Friday 17:00) / on-demand
**Boundary:** No company-level financial forecasting. No product decisions.

| **Trigger** | **Execution** | **Quality** |
|-|-|-|
| ✅ Daily briefing automatic; weekly cron pending | ⚠️ Weekly pipeline summary skill not yet built; on-demand review works ✅ | ⚠️ No conversion trend or forecast vs target output yet |

**Skills** *(building blocks):* `daily-briefing` · `reviewing-sales-pipeline` · `reviewing-partnership-pipeline` · *(pending)* `weekly-pipeline-review`
**Cron:** → `daily-briefing` (08:00 daily) · → *(pending)* `weekly-pipeline-review` (Friday 17:00)

---

### Partner Success *(out of scope for Leo)*

Leo's boundary ends at **Partnership Signed**. Once a partner reaches `SIGNED` status, Leo creates a handoff task and notifies the Partner Success Agent.

Partner Success — enablement, joint go-to-market, revenue tracking, health monitoring — belongs to a dedicated agent with a different north star: **retention and amplification**, not progression.

**Partner Success Agent:** *Pending — agent not yet defined.*

---

## Status Overview

| **Capability** | **Funnel Stage** | **Trigger** | **Execution** | **Quality** |
|-|-|-|-|-|
| C1 Outbound Prospecting & Account Intelligence | ToFu | ⚠️ / ✅ | ⚠️ | ⚠️ |
| C2 Lead Nurturing | MoFu | ⚠️ / ✅ | ⚠️ | ⚠️ |
| C3 Pipeline Progressing | MoFu → BoFu | ⚠️ / ✅ | ✅ | ⚠️ |
| C4 Partnership Progressing | MoFu → BoFu | ✅ / ⚠️ | ✅ | ⚠️ |
| C5 Pipeline Health Monitoring | Cross-funnel | ✅ / ⚠️ | ⚠️ | ⚠️ |

**Pending skills:** `lead-list-triage` (C1) · `mql-outreach` (C2) · `outreach-sequence-check` cron (C2) · `weekly-pipeline-review` (C5)

---

## Supporting Skills

| **Skill** | **What It Does** | **Used By** |
|-|-|-|
| `follow-up-email` | Draft a follow-up message based on deal/partner context and last interaction | C2, C3, C4 |
| `generating-quotations` | Generate quotation document from Opportunity and Company data | C3 |
| `generating-invoices` | Generate invoice after contract is signed | C3 |
| `pitch-deck` | Structure presentation materials for a prospect or partner meeting | C3, C4 |
| `deal-advisory` | Deep diagnosis of a stalled or stuck opportunity — history analysis + recovery plan | C3 |
| `meeting-prep` | Generate contextual brief before any scheduled meeting | C3, C4 |
| `task-management` | Identify all actionable work items from an engagement, create Tasks with full context | C3, C4 |

---

## Tools

| **Tool** | **Purpose** | **Used By** |
|-|-|-|
| Twenty CRM (self-hosted) | Source of truth — Companies, People, Opportunities, Partnerships, Engagements, Tasks, Notes | All |
| GBrain | Long-term knowledge — narratives, company intel, partner history, relationship context | C1, C3, C4 |
| Web Search (Tavily) | Company research, list triage, account enrichment | C1, C2 |
| Lark IM | Delivering briefs, alerts, and draft batches to the sales rep | All |
| Lark Docs / Drive | Quotation and proposal document generation and storage | C3 |
| Hermes Cron | Scheduling and running automated jobs | All |

---

## What Leo Does Not Do

- Inbound lead generation → Maya
- Sourcing raw prospect lists — provided by the sales rep or Maya
- Sending external communications autonomously — Leo drafts, human sends
- Content creation (copywriting, blog posts, collateral) → Content agent
- Product decisions → Product team
- Post-sale customer support → Support agent
- Partner success post-signing → Partner Success Agent (pending)
- Company-level financial forecasting → Finance

---

## Design Principles

**Leo Owns the Outbound Motion and the Full Pipeline**
From prospect list to closed customer. C1 builds context, C2 warms up, C3 and C4 drive to close, C5 monitors the whole.

**Three Entry Paths, One Pipeline**
Maya inbound, Leo outbound, and Human outbound all feed the same CRM. Once a prospect enters CRM, Leo owns the progression regardless of source.

**Enrichment Depth Matches Intent**
Not every prospect deserves the same research investment. Low-intent cold list = basic enrichment. High-intent enquiry = deep enrichment. Human-introduced contact = ask for more context.

**Discard, Don't Accumulate**
Names that don't pass triage are discarded entirely. CRM is a working tool, not a contact database. Every record must have a reason to be there.

**OPT_OUT Is Permanent Until Overridden**
Persons who say do not contact stay in CRM for record-keeping but are excluded from all enrichment, outreach, and pipeline views. Only a human can override.

**C3 and C4 Are the Same Flow**
Opportunity progression and Partnership progression share identical logic — accept any input, extract summary + tasks, update CRM, sync GBrain. One pattern, two objects, two end goals.

**Every Cron Maps to a Skill**
Cron jobs are triggers only. All logic lives in the skill. Any Capability can be invoked manually at any time with identical behaviour.

**Drafts, Not Sends**
Leo never sends external communications autonomously. Every outbound message is prepared as a draft for human confirmation.

**GBrain Is Always Updated**
Every new Company, Person, and Engagement is automatically reflected in GBrain. Twenty CRM stores current facts. GBrain accumulates the narrative over time.
