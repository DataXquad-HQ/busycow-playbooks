# Maya — Build List

**Last Updated:** 2026-06-12

This is the complete list of everything Maya needs to operate at full capability. Items are grouped by capability. Status is flagged on every item.

- ✅ Built & working
- ⚠️ Partially built / needs config
- 🔴 Not built yet

---

## Foundation Layer — Market Intelligence

| # | Item | Type | Status | Notes |
|---|---|---|---|---|
| F1 | ICP Profiles table in Lark Base | Data structure | 🔴 Not built | Needs: ICP definition session with Hunter/Kevin first |
| F2 | Content Calendar table in Lark Base | Data structure | 🔴 Not built | Tracks planned + published content per week |
| F3 | `market-scan` cron job | Cron | 🔴 Not built | Monday 07:00 — web scan of target segments + competitors |
| F4 | `market-scan` skill | Skill | 🔴 Not built | Weekly research sweep; writes output to GBrain |
| F5 | GBrain market map pages | Knowledge | 🔴 Not built | Market segments, competitor intel, ICP narratives — all empty |
| F6 | ICP definition | Strategic input | 🔴 Not done | **Blocker for everything else.** Needs 30-min session with Hunter/Kevin |

---

## Capability 1 — Long-Form Content

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

## Capability 2 — Social Media Presence

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

## Capability 3 — Lead Capture

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

## Summary

| Status | Count |
|---|---|
| ✅ Built & working | 10 |
| ⚠️ Partial / needs config | 1 |
| 🔴 Not built | 23 |

---

## Recommended Build Order

Build in dependency order — don't build downstream items before their blockers are cleared.

### Phase 0 — Unblock everything (do this first)
1. **F6** — ICP definition session with Hunter/Kevin ← everything depends on this
2. **C2-9** — Confirm GeoKernel LinkedIn page exists

### Phase 1 — Data structures (foundation before content)
3. **F1** — ICP Profiles table in Lark Base
4. **F2 / C1-12** — Content Calendar table in Lark Base
5. **C3-8** — Lead Capture Log table in Lark Base CRM

### Phase 2 — Content engine (CAP 1)
6. **C1-13 / C1-14** — Create Medium + Substack accounts for GeoKernel
7. **C1-7** — `medium-publish` skill
8. **C1-9** — Choose and set up newsletter platform
9. **C1-8** — `substack-publish` skill
10. **C1-10** — `newsletter-send` skill
11. **C1-11** — `content-queue-monday` cron job

### Phase 3 — Social engine (CAP 2)
12. **C2-4** — Install Postiz on VM
13. **C2-5 / C2-6** — Connect LinkedIn + X to Postiz
14. **C2-7** — `postiz` skill
15. **C2-8** — `linkedin-post` skill
16. **C2-10** — `social-queue-monday` cron job
17. **C2-11** — Social DM monitoring

### Phase 4 — Lead capture (CAP 3)
18. **C3-5** — Lead capture form on website
19. **C3-6** — `form-to-crm` skill
20. **C3-7** — `newsletter-subscriber-sync` skill
21. **C3-9** — First ICP-targeted landing page
22. **C3-10** — First lead magnet asset

### Phase 5 — Intelligence automation (Foundation)
23. **F4** — `market-scan` skill
24. **F3** — `market-scan` cron job (Monday 07:00)
25. **F5** — Populate GBrain market map pages
