---
name: packaging-to-github
description: Use when publishing internal skills, setup configs, or playbooks to the busycow-playbooks GitHub repo — so clients can install them with a single URL. Covers universalization (strip internal data), sanitization scan (no tokens/IDs/names), repo structure, and push workflow.
triggers:
  - "push to github"
  - "publish playbook"
  - "package for clients"
  - "put on github"
  - "幫客戶裝"
  - "publish skill"
  - "push 上去"
version: "2.0"
author: BusyCow
---

# Packaging to GitHub

Use this when publishing anything to `busycow-playbooks` — setup configs, core skills, or business playbooks — so clients can install with a single URL.

## Repo
```
https://github.com/DataXquad-HQ/busycow-playbooks
Local: ~/busycow-playbooks
```

## Repo Structure
```
busycow-playbooks/
├── README.md                    ← Total overview + Prerequisites
├── setup/                       ← Stack config (SOUL.md + MEMORY templates)
│   ├── SETUP.md
│   └── templates/
│       ├── SOUL.md
│       └── MEMORY.md
├── core/                        ← Core Playbook (install before business playbooks)
│   ├── README.md
│   ├── SETUP.md
│   └── skills/
│       ├── managing-skills.md
│       ├── managing-cron-jobs.md
│       ├── maintaining-gbrain.md
│       ├── maintaining-memory.md
│       └── capturing-to-gbrain.md
└── playbooks/
    ├── sales/
    │   ├── README.md
    │   ├── SETUP.md
    │   └── skills/
    │       ├── capturing-sales-intel.md
    │       ├── logging-sales-activities.md
    │       ├── managing-sales-pipeline.md
    │       ├── managing-partnership-pipeline.md
    │       ├── reviewing-sales-pipeline.md
    │       ├── enriching-leads.md
    │       └── generating-quotations.md
    └── internal-ops/
        ├── README.md
        ├── SETUP.md
        └── skills/
            ├── managing-tasks.md
            ├── reviewing-tasks.md
            ├── planning-next-actions.md
            ├── generating-task-briefing.md
            └── auditing-tasks.md
```

## Installation Order (client-facing)
```
Step 1: setup/SETUP.md      → writes SOUL.md + Memory skeleton
Step 2: core/SETUP.md       → builds Hermes Registry Base + installs core skills
Step 3: playbooks/X/SETUP.md → builds business Base + installs business skills
```

---

## Workflow

### Step 1 — Read source content
Use `skill_view(name)` or `read_file` to get the internal skill/config content.

### Step 2 — Universalize
For every file, apply these rules:

| Remove | Replace with |
|--------|-------------|
| Hardcoded Lark base/table IDs | `{{TABLE_NAME_TABLE_ID}}` placeholder |
| Hardcoded field IDs | `{{FIELD_ID}}` or remove |
| API tokens (Lark, Notion, Google) | `{{API_TOKEN}}` or remove |
| Google Doc / Drive IDs | `{{GOOGLE_DOC_TEMPLATE_ID}}` |
| IP addresses | `{{SERVER_IP}}` |
| Internal product names ([Product], [Product], [Product]) | `[Product]` or `[your product lines]` |
| Specific client/partner names ([Client], [Client], etc.) | `[Client]` or `[Partner]` |
| Personal names / usernames (Hunter, the_owner) | `the owner` or omit |
| Personal file paths (/home/username) | `~` |
| Internal business logic (billing entity rules, commission %) | Generic equivalent or remove |
| Company name BusyCow | BusyCow (as publisher) |

### Step 3 — Sanitization scan (mandatory before push)

Run this Python scan against every file being pushed:

```python
import re, os

CONFIDENTIAL_PATTERNS = [
    # Lark-specific IDs (prefixed — low false-positive risk)
    (r'tbl[A-Za-z0-9]{8,}', 'hardcoded table ID'),
    (r'fld[A-Za-z0-9]{8,}', 'hardcoded field ID'),
    (r'ou_[a-z0-9]{20,}', 'hardcoded open_id'),
    (r'oc_[a-z0-9]{20,}', 'hardcoded chat ID'),
    (r'ntn_[A-Za-z0-9]+', 'Notion token'),
    (r'LARK_APP_(?:ID|SECRET)=[^\n{]', 'Lark credential'),
    # Known internal app tokens (add new ones here as they appear)
    (r'MtvNbgCH[A-Za-z0-9]+', 'hardcoded Lark app token (Sales CRM)'),
    (r'SfMJbDOB[A-Za-z0-9]+', 'hardcoded Lark app token (Task Tracker)'),
    # Internal data
    (r'\bDataXquad\b(?!-HQ)', 'company name BusyCow'),
    (r'\b([Product]|[Product]|[Product])\b', 'internal product name'),
    (r'/home/[a-z_]+', 'personal path'),
    (r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b', 'IP address'),
    # Personal info
    (r'[a-z]+@dataxquad\.com', 'internal email'),
    (r'\bhunter_lin\b', 'personal username'),
    # Internal partner/client names
    (r'\b([Client]|[Client]|[Client]|HeadWorker|VoiceBot)\b', 'internal partner name'),
]

def scan(directory):
    issues = {}
    for root, _, files in os.walk(os.path.expanduser(directory)):
        if '.git' in root:
            continue
        for filename in files:
            if not filename.endswith('.md'):
                continue
            filepath = os.path.join(root, filename)
            with open(filepath) as f:
                content = f.read()
            file_issues = []
            for pattern, label in CONFIDENTIAL_PATTERNS:
                matches = re.findall(pattern, content)
                if matches:
                    file_issues.append(f"  ⚠️  {label}: {list(set(matches))[:3]}")
            if file_issues:
                issues[filepath] = file_issues
    return issues

issues = scan("~/busycow-playbooks")
if not issues:
    print("✅ All clean — safe to push.")
else:
    for f, problems in issues.items():
        print(f"📄 {f}")
        for p in problems:
            print(p)
```

**Known false positives to ignore:**
- `LARK_APP_SECRET=***` in README Step 0c — this is example text, not a real secret
- Any English word starting with `rec` (e.g. `recommendation`, `record`) — NOT a Lark record ID; real Lark record IDs look like `recvjdoVbmQane` (14+ chars, mixed case). The pattern above avoids this by not scanning for bare `rec...` patterns.

**Do not push if scan returns any real issues. Fix first.**

**Do not push if scan returns any issues. Fix first.**

### Step 4 — Write files to repo
Save each universalized file to the correct location under `~/busycow-playbooks/`.

### Step 5 — Push
```bash
cd ~/busycow-playbooks
git add -A
git commit -m "feat/refactor: [description of what was added or changed]"
git push origin main
```

### Step 6 — Confirm client URLs
After pushing, the client install URLs are:
```
https://raw.githubusercontent.com/DataXquad-HQ/busycow-playbooks/main/[path]/SETUP.md
```

---

## SETUP.md Format (agent-executable)

Every SETUP.md must be written as **step-by-step agent instructions**, not human prose.
The agent reads the URL and executes each step automatically.

Required sections:
1. What this setup creates (bullet list)
2. Numbered steps with exact API calls or fetch URLs
3. Verify section (how to confirm it worked)
4. Next step (what to run next)

Must be **idempotent** — running twice should not corrupt the workspace.

---

## Parallelising Large Batches

**Do NOT use `delegate_task` for bulk skill packaging.** Subagents time out (~80s) when given many large skill files to read and process — they complete the reads but never write anything before hitting the timeout. This was confirmed in a 37-skill push (May 2026): all 3 parallel subagents interrupted after reading, zero files written.

**Use `execute_code` with a batch Python script instead.** It reads all source files, applies universalization rules via regex, writes all output files, and completes in under 1 second — regardless of how many skills are being packaged.

Pattern:
```python
# In execute_code — reads all skills, universalizes, writes to repo in one pass
skill_mapping = {
    'core/skills/hermes-agent.md': 'autonomous-ai-agents/hermes-agent/SKILL.md',
    'playbooks/sales/skills/capturing-sales-intel.md': 'productivity/capturing-sales-intel/SKILL.md',
    # ... all mappings
}

for dest_rel, src_rel in skill_mapping.items():
    content = read_skill(os.path.join(SKILLS_ROOT, src_rel))
    content = universalize(content)  # regex substitutions
    os.makedirs(os.path.dirname(dest_path), exist_ok=True)
    with open(dest_path, 'w') as f:
        f.write(content)
```

Then run the sanitization scan (also in `execute_code`), fix any stragglers, and push.

---

## Pitfalls

- Run sanitization scan **every time**, even for small edits — easy to miss one field
- `{{PLACEHOLDER}}` format for all dynamic values — don't leave bare TODO comments
- SETUP.md must work as a **standalone agent instruction** — don't assume the agent has context from the README
- Raw GitHub URLs only for client install — `raw.githubusercontent.com`, not the HTML page
- Don't add Notion-based variants or other stacks — one stack only (Hermes + Lark + GBrain)
- After push, verify with: `curl -s [raw URL] | head -5` to confirm file is accessible
- Sanitization scan: use `os.path.expanduser()` on the directory path — bare `~` doesn't expand in Python's `os.walk`
- Internal server paths appear in two forms: `/home/the_owner/` AND `~/.hermes/` — add both to the universalize regex, or use a greedy pattern: `r'~/.hermes[^\s]*'` → `~/.hermes`
- Scan false positives: `LARK_APP_SECRET=***` in README is safe (example text). English words starting with `rec` (recommendation, record) are NOT Lark record IDs — don't add a bare `rec[A-Za-z0-9]{8,}` pattern to the scanner
