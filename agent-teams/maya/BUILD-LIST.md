# Maya — Build List

**Last Updated:** 2026-06-12

This is the complete list of everything Maya needs to operate at full capability across **all business lines**.

**Business Lines:** BusyCow · GeoKernel · AquaOptima · TRACI · Distify · DataXquad

Maya's capabilities apply to every business line. Each line needs its own: website, blog, social accounts, newsletter, ICP definition, and lead capture setup. Shared infrastructure (tools, skills, crons) is built once and reused.

- ✅ Built & working
- ⚠️ Partially built / needs config
- 🔴 Not built yet

---

## Per-Business-Line Assets

These must be set up **separately for each business line**. They cannot be shared.

### Websites & Blogs

| Business Line | Website | Blog | Status | Notes |
|---|---|---|---|---|
| GeoKernel | geokernel.com (or similar) | blog.geokernel.com | ⚠️ Ghost CMS live | Confirm domain + blog live |
| BusyCow | busycow.ai (or similar) | — | 🔴 Unknown | **Confirm with Hunter/Kevin** |
| AquaOptima | — | — | 🔴 Unknown | **Confirm with Hunter/Kevin** |
| TRACI | — | — | 🔴 Unknown | **Confirm with Hunter/Kevin** |
| Distify | — | — | 🔴 Unknown | **Confirm with Hunter/Kevin** |
| DataXquad | dataxquad.com | blog.dataxquad.com | ✅ Ghost CMS live | |

### LinkedIn Company Pages

| Business Line | LinkedIn Page | Status | Notes |
|---|---|---|---|
| GeoKernel | — | 🔴 Unknown | **Confirm with Hunter/Kevin** |
| BusyCow | — | 🔴 Unknown | |
| AquaOptima | — | 🔴 Unknown | |
| TRACI | — | 🔴 Unknown | |
| Distify | — | 🔴 Unknown | |
| DataXquad | — | 🔴 Unknown | |

### Newsletter / Substack Publications

| Business Line | Newsletter | Status | Notes |
|---|---|---|---|
| GeoKernel | — | 🔴 Not set up | Decide platform first (see C1-9) |
| BusyCow | — | 🔴 Not set up | |
| AquaOptima | — | 🔴 Not set up | |
| TRACI | — | 🔴 Not set up | |
| Distify | — | 🔴 Not set up | |
| DataXquad | — | 🔴 Not set up | |

### Medium Publications

| Business Line | Medium Publication | Status |
|---|---|---|
| GeoKernel | — | 🔴 Not set up |
| BusyCow | — | 🔴 Not set up |
| AquaOptima | — | 🔴 Not set up |
| TRACI | — | 🔴 Not set up |
| Distify | — | 🔴 Not set up |
| DataXquad | — | 🔴 Not set up |

### ICP Definitions

| Business Line | ICP Defined | Status | Notes |
|---|---|---|---|
| GeoKernel | — | 🔴 Not done | **Blocker for all content** |
| BusyCow | — | 🔴 Not done | |
| AquaOptima | — | 🔴 Not done | |
| TRACI | — | 🔴 Not done | |
| Distify | — | 🔴 Not done | |
| DataXquad | — | 🔴 Not done | |

### Lead Capture Forms (per website)

| Business Line | Form on Website | Routes to CRM | Status |
|---|---|---|---|
| GeoKernel | — | — | 🔴 Not built |
| BusyCow | — | — | 🔴 Not built |
| AquaOptima | — | — | 🔴 Not built |
| TRACI | — | — | 🔴 Not built |
| Distify | — | — | 🔴 Not built |
| DataXquad | — | — | 🔴 Not built |

---

---

## Shared Infrastructure

These are built **once** and reused across all business lines.

---

### Foundation Layer — Market Intelligence

| # | Item | Type | Status | Notes |
|---|---|---|---|---|
| F1 | ICP Profiles table in Lark Base | Data structure | 🔴 Not built | Needs: ICP definition session with Hunter/Kevin first |
| F2 | Content Calendar table in Lark Base | Data structure | 🔴 Not built | Tracks planned + published content per week |
| F3 | `market-scan` cron job | Cron | 🔴 Not built | Monday 07:00 — web scan of target segments + competitors |
| F4 | `market-scan` skill | Skill | 🔴 Not built | Weekly research sweep; writes output to GBrain |
| F5 | GBrain market map pages | Knowledge | 🔴 Not built | Market segments, competitor intel, ICP narratives — all empty |
| F6 | ICP definition | Strategic input | 🔴 Not done | **Blocker for everything else.** Needs 30-min session with Hunter/Kevin |

---

### Capability 1 — Long-Form Content

| # | Item | Type | Status | Notes |
|---|---|---|---|---|
| C1-1 | `writing-blog-post` skill | Skill | ✅ Built | Working |
| C1-2 | `humanizer` skill | Skill | ✅ Built | Working |
| C1-3 | `baoyu-infographic` skill | Skill | ✅ Built | Working |
| C1-4 | `imagen-3` image generation | Skill / Tool | ✅ Built | Google AI Studio API key available |
| C1-5 | `youtube-content` skill | Skill | ✅ Built | Working |
| C1-6 | Ghost CMS blog publish | Tool | ✅ Built | `astro-ghost-vercel-website` skill covers this |
| C1-7 | `medium-publish` skill | Skill | 🔴 Not built | Medium API integration — publish/syndicate posts |
| C1-8 | `substack-publish` skill | Skill | 🔴 Not built | Substack API or RSS-based syndication |
| C1-9 | Newsletter platform setup | Tool | 🔴 Not built | Decide platform (Substack vs Beehiiv vs self-hosted). **Decision needed.** |
| C1-10 | `newsletter-send` skill | Skill | 🔴 Not built | Compile + send newsletter; depends on C1-9 |
| C1-11 | `content-queue-monday` cron job | Cron | 🔴 Not built | Monday 09:00 — plan 2–3 pieces for the week |
| C1-12 | Content Calendar in Lark Base | Data structure | 🔴 Not built | Shared with F2 |
| C1-13 | Medium account for GeoKernel | Account | 🔴 Not set up | Need account created + API key |
| C1-14 | Substack for GeoKernel | Account | 🔴 Not set up | Need publication created |

---

### Capability 2 — Social Media Presence

| # | Item | Type | Status | Notes |
|---|---|---|---|---|
| C2-1 | `humanizer` skill | Skill | ✅ Built | Working |
| C2-2 | `imagen-3` image generation | Skill / Tool | ✅ Built | Working |
| C2-3 | `xurl` skill | Skill | ✅ Built | X/Twitter posting working |
| C2-4 | Postiz — install & setup | Tool | 🔴 Not built | Self-hosted via Docker. **Needs install on VM.** |
| C2-5 | Postiz — LinkedIn account connected | Config | 🔴 Not built | Depends on C2-4 |
| C2-6 | Postiz — X/Twitter account connected | Config | 🔴 Not built | Depends on C2-4 |
| C2-7 | `postiz` skill | Skill | 🔴 Not built | Maya interacts with Postiz API to queue posts |
| C2-8 | `linkedin-post` skill | Skill | 🔴 Not built | LinkedIn-native formatting + publish via Postiz or LinkedIn API |
| C2-9 | GeoKernel LinkedIn page | Account | ⚠️ Unknown | Does GeoKernel have a LinkedIn company page? **Confirm with Hunter/Kevin.** |
| C2-10 | `social-queue-monday` cron job | Cron | 🔴 Not built | Monday 10:00 — queue approved content for the week |
| C2-11 | Social DM monitoring | Capability | 🔴 Not built | Monitor LinkedIn + X DMs for inbound signals; flag to Lark IM |

---

### Capability 3 — Lead Capture

| # | Item | Type | Status | Notes |
|---|---|---|---|---|
| C3-1 | `astro-ghost-vercel-website` skill | Skill | ✅ Built | Website pages + Ghost CMS working |
| C3-2 | `baoyu-infographic` skill | Skill | ✅ Built | Lead magnet design working |
| C3-3 | `humanizer` skill | Skill | ✅ Built | Working |
| C3-4 | `imagen-3` image generation | Skill / Tool | ✅ Built | Working |
| C3-5 | Lead capture form on website | Config | 🔴 Not built | Ghost CMS form or embedded Tally/Typeform. Needs setup + test. |
| C3-6 | `form-to-crm` skill | Skill | 🔴 Not built | Route form submission → Lark Base CRM with source tag |
| C3-7 | `newsletter-subscriber-sync` skill | Skill | 🔴 Not built | Sync newsletter subscribers → CRM contact list; depends on C1-9 |
| C3-8 | Lead Capture Log in Lark Base CRM | Data structure | 🔴 Not built | Table to receive all inbound leads with source tag |
| C3-9 | First landing page (ICP-targeted) | Content | 🔴 Not built | Depends on F6 (ICP definition) |
| C3-10 | First lead magnet asset | Content | 🔴 Not built | Depends on F6 (ICP definition) |

---

### Summary

| Status | Count |
|---|---|
| ✅ Built & working | 10 |
| ⚠️ Partial / needs config | 1 |
| 🔴 Not built | 23 |

---

## Build Order

Build in dependency order. Shared infrastructure first, then replicate per business line.

### Phase 0 — Confirm reality (do this first, no building until done)
1. ICP definition session — all 6 business lines. 60-min session with Hunter/Kevin. **Everything downstream is blocked without this.**
2. Confirm which websites/blogs already exist per BL
3. Confirm which LinkedIn pages already exist per BL
4. Decide newsletter platform (Substack vs Beehiiv vs self-hosted) — one decision applies to all BLs

### Phase 1 — Shared infrastructure (built once)
5. **F1** — ICP Profiles table in Lark Base (one table, BL column)
6. **F2 / C1-12** — Content Calendar table in Lark Base (one table, BL column)
7. **C3-8** — Lead Capture Log in CRM (one table, BL + source columns)
8. **C2-4** — Install Postiz on VM
9. **C1-7** — `medium-publish` skill
10. **C1-8** — `substack-publish` skill
11. **C1-10** — `newsletter-send` skill
12. **C2-7** — `postiz` skill
13. **C2-8** — `linkedin-post` skill
14. **C3-6** — `form-to-crm` skill
15. **C3-7** — `newsletter-subscriber-sync` skill
16. **F4** — `market-scan` skill
17. **C1-11** — `content-queue-monday` cron job
18. **C2-10** — `social-queue-monday` cron job
19. **F3** — `market-scan` cron job

### Phase 2 — Per-BL setup (repeat for each business line)
For each of: **GeoKernel · BusyCow · AquaOptima · TRACI · Distify · DataXquad**

- [ ] Website confirmed/built
- [ ] Blog on Ghost CMS confirmed/built
- [ ] Medium publication created + API key
- [ ] Newsletter publication created (platform decided in Phase 0)
- [ ] LinkedIn company page confirmed/created + connected to Postiz
- [ ] X account confirmed/created + connected to Postiz
- [ ] Lead capture form on website
- [ ] First ICP-targeted landing page (depends on Phase 0 ICP definition)
- [ ] GBrain market map page created for this BL
- [ ] First lead magnet asset

**Priority order for Phase 2:** Start with the BL closest to revenue — confirm with Hunter/Kevin which BL to activate first.
