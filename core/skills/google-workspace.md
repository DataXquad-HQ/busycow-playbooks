---
name: google-workspace
description: >
  Use Gmail, Google Calendar, Drive, Sheets, and Docs via the gws CLI.
  Use when user asks to send an email, check calendar, read/write a Google Doc or Sheet,
  search Drive, or manage Google contacts. Requires gws binary and valid OAuth token.
version: 2.0.0
author: Nous Research
license: MIT
required_credential_files:
  - path: google_token.json
    description: Google OAuth2 token (created by setup script)
  - path: google_client_secret.json
    description: Google OAuth2 client credentials (downloaded from Google Cloud Console)
metadata:
  hermes:
    tags: [Google, Gmail, Calendar, Drive, Sheets, Docs, Contacts, Email, OAuth, gws]
    homepage: https://github.com/NousResearch/hermes-agent
    related_skills: [himalaya]
---

# Google Workspace

Gmail, Calendar, Drive, Contacts, Sheets, and Docs — powered by `gws` (Google's official Rust CLI). The skill provides a backward-compatible Python wrapper that handles OAuth token refresh and delegates to `gws`.

## Architecture

```
google_api.py  →  gws_bridge.py  →  gws CLI
(argparse compat)  (token refresh)    (Google APIs)
```

- `setup.py` handles OAuth2 (headless-compatible, works on CLI/Telegram/Discord)
- `gws_bridge.py` refreshes the Hermes token and injects it into `gws` via `GOOGLE_WORKSPACE_CLI_TOKEN`
- `google_api.py` provides the same CLI interface as v1 but delegates to `gws`

## References

- `references/gmail-search-syntax.md` — Gmail search operators (is:unread, from:, newer_than:, etc.)
- `references/presentation-reframing.md` — Editorial pattern for converting a company pitch deck into an observation/sharing-tone talk (structure, language moves, audience calibration, Google Docs editing sequence).

## Scripts

- `scripts/setup.py` — OAuth2 setup (run once to authorize)
- `scripts/gws_bridge.py` — Token refresh bridge to gws CLI
- `scripts/google_api.py` — Backward-compatible API wrapper (delegates to gws)

## Prerequisites

Install `gws`:

```bash
cargo install google-workspace-cli
# or via npm (recommended, downloads prebuilt binary):
npm install -g @googleworkspace/cli
# or via Homebrew:
brew install googleworkspace-cli
```

Verify: `gws --version`

## First-Time Setup

The setup is fully non-interactive — you drive it step by step so it works
on CLI, Telegram, Discord, or any platform.

Define a shorthand first:

```bash
HERMES_HOME="${HERMES_HOME:-$HOME/.hermes}"
GWORKSPACE_SKILL_DIR="$HERMES_HOME/skills/productivity/google-workspace"
PYTHON_BIN="${HERMES_PYTHON:-python3}"
if [ -x "$HERMES_HOME/hermes-agent/venv/bin/python" ]; then
  PYTHON_BIN="$HERMES_HOME/hermes-agent/venv/bin/python"
fi
GSETUP="$PYTHON_BIN $GWORKSPACE_SKILL_DIR/scripts/setup.py"
```

### Step 0: Check if already set up

```bash
$GSETUP --check
```

If it prints `AUTHENTICATED`, skip to Usage — setup is already done.

> **Note on actual file location (busycow-hermes VM):** The credential files are stored at `~/.hermes/` (the persistent data disk), NOT inside the skill directory. The setup script auto-detects this via `HERMES_HOME` or falls back to the data disk path. To copy credentials to another VM:
> ```bash
> # Source files are at:
> ~/.hermes/google_token.json
> ~/.hermes/google_client_secret.json
> # Verify with:
> python3 ~/.hermes/skills/productivity/google-workspace/scripts/setup.py --check
> # → prints: AUTHENTICATED: Token valid at ~/.hermes/google_token.json
> ```

### Step 1: Triage — ask the user what they need

**Question 1: "What Google services do you need? Just email, or also
Calendar/Drive/Sheets/Docs?"**

- **Email only** → Use the `himalaya` skill instead — simpler setup.
- **Calendar, Drive, Sheets, Docs (or email + these)** → Continue below.

**Partial scopes**: Users can authorize only a subset of services. The setup
script accepts partial scopes and warns about missing ones.

**Question 2: "Does your Google account use Advanced Protection?"**

- **No / Not sure** → Normal setup.
- **Yes** → Workspace admin must add the OAuth client ID to allowed apps first.

### Step 2: Create OAuth credentials (one-time, ~5 minutes)

Tell the user:

> 1. Go to https://console.cloud.google.com/apis/credentials
> 2. Create a project (or use an existing one)
> 3. Enable the APIs you need (Gmail, Calendar, Drive, Sheets, Docs, People)
> 4. Credentials → Create Credentials → OAuth 2.0 Client ID → Desktop app
> 5. Download JSON and tell me the file path

```bash
$GSETUP --client-secret /path/to/client_secret.json
```

### Step 3: Get authorization URL

```bash
$GSETUP --auth-url
```

Send the URL to the user. After authorizing, they paste back the redirect URL or code.

### Step 4: Exchange the code

```bash
$GSETUP --auth-code "THE_URL_OR_CODE_THE_USER_PASTED"
```

### Step 5: Verify

```bash
$GSETUP --check
```

Should print `AUTHENTICATED`. Token refreshes automatically from now on.

## Usage

All commands go through the API script:

```bash
HERMES_HOME="${HERMES_HOME:-$HOME/.hermes}"
GWORKSPACE_SKILL_DIR="$HERMES_HOME/skills/productivity/google-workspace"
PYTHON_BIN="${HERMES_PYTHON:-python3}"
if [ -x "$HERMES_HOME/hermes-agent/venv/bin/python" ]; then
  PYTHON_BIN="$HERMES_HOME/hermes-agent/venv/bin/python"
fi
GAPI="$PYTHON_BIN $GWORKSPACE_SKILL_DIR/scripts/google_api.py"
```

### Gmail

```bash
$GAPI gmail search "is:unread" --max 10
$GAPI gmail get MESSAGE_ID
$GAPI gmail send --to user@example.com --subject "Hello" --body "Message text"
$GAPI gmail send --to user@example.com --subject "Report" --body "<h1>Q4</h1>" --html
$GAPI gmail reply MESSAGE_ID --body "Thanks, that works for me."
$GAPI gmail labels
$GAPI gmail modify MESSAGE_ID --add-labels LABEL_ID
```

### Calendar

```bash
$GAPI calendar list
$GAPI calendar create --summary "Standup" --start 2026-03-01T10:00:00+01:00 --end 2026-03-01T10:30:00+01:00
$GAPI calendar create --summary "Review" --start ... --end ... --attendees "alice@co.com,bob@co.com"
$GAPI calendar delete EVENT_ID
```

### Drive

```bash
$GAPI drive search "quarterly report" --max 10
$GAPI drive search "mimeType='application/pdf'" --raw-query --max 5
```

#### Shared Drive operations (requires direct REST API — gws CLI does not support Shared Drives)

Use `urllib.request` directly with the Drive v3 API. Key difference from My Drive: every request needs
`supportsAllDrives=true`, `includeItemsFromAllDrives=true`, and `corpora=drive` + `driveId`.

```python
import sys, os, json, urllib.request, urllib.parse, time
sys.path.insert(0, f"{os.environ.get('HERMES_HOME', os.path.expanduser('~/.hermes'))}/skills/productivity/google-workspace/scripts")
from gws_bridge import get_valid_token

token = get_valid_token()
DRIVE_ID = "{{SHARED_DRIVE_ID}}"  # last segment of the Shared Drive URL

def gapi(method, path, body=None, params_dict=None):
    base = f"https://www.googleapis.com/drive/v3/{path}"
    if params_dict:
        base += "?" + urllib.parse.urlencode(params_dict)  # MUST use urlencode — spaces in q= cause InvalidURL
    data = json.dumps(body).encode() if body else None
    headers = {"Authorization": f"Bearer {token}"}
    if body:
        headers["Content-Type"] = "application/json"
    req = urllib.request.Request(base, data=data, headers=headers, method=method)
    with urllib.request.urlopen(req, timeout=15) as r:
        return json.loads(r.read())

def list_folder(folder_id):
    """List contents of a folder in the Shared Drive."""
    params = {
        "driveId": DRIVE_ID,
        "includeItemsFromAllDrives": "true",
        "supportsAllDrives": "true",
        "corpora": "drive",
        "q": f"'{folder_id}' in parents",
        "fields": "files(id,name,mimeType,parents)",
        "pageSize": "100",
        "orderBy": "name"
    }
    return gapi("GET", "files", params_dict=params).get("files", [])

def create_folder(name, parent_id):
    r = gapi("POST", "files", {"name": name, "mimeType": "application/vnd.google-apps.folder", "parents": [parent_id]},
             {"supportsAllDrives": "true"})
    return r["id"]

def move(file_id, new_parent_id, old_parent_id):
    """Move a file/folder within Shared Drive — must pass both addParents and removeParents."""
    return gapi("PATCH", f"files/{file_id}", {}, {
        "addParents": new_parent_id,
        "removeParents": old_parent_id,
        "supportsAllDrives": "true",
        "fields": "id,name"
    })

def rename(file_id, new_name):
    return gapi("PATCH", f"files/{file_id}", {"name": new_name}, {"supportsAllDrives": "true"})

# List all Shared Drives the OAuth account has access to:
drives = gapi("GET", "drives", {"pageSize": "20"})
for d in drives.get("drives", []):
    print(d["name"], d["id"])
```

**Moving orphaned My Drive docs INTO a Shared Drive:**

`PATCH /drive/v3/files/{id}?addParents=...` with `canMoveItemIntoTeamDrive=false` will return 403 even with editor permission. Orphaned docs (files with empty `parents: []`) that live in another user's My Drive cannot be moved into a Shared Drive by the OAuth account — only the owner can. The correct pattern is to **copy** instead:

```python
# Copy the file into the target Shared Drive folder
result = gapi("POST", f"files/{doc_id}/copy",
    {"name": original_name, "parents": [TARGET_FOLDER_ID]},
    {"supportsAllDrives": "true"}
)
new_id = result["id"]
# Original stays in owner's My Drive — they can delete it themselves
```

Diagnosis: check `capabilities.canMoveItemIntoTeamDrive` on the file metadata. If `false`, copy is the only path.

**Critical pitfalls for Shared Drive API:**
- ❌ `urllib.request` raises `InvalidURL: URL can't contain control characters` if you build `?q='folder_id' in parents` by string concatenation — spaces in `in parents` cause it. **Always use `urllib.parse.urlencode(params_dict)` to build the query string.**
- ❌ Using wrong OAuth app ID → `403 Forbidden` on write ops. Shared Drive writes require the app that **owns** the Drive. Use `cli_a97bd21895f89e18` (Hermes-BusyCow) for the BusyCow Shared Drive, NOT `cli_a96d2de81f3a1e17` (lark-mcp app).
- ❌ `move()` without `removeParents` leaves the file in both locations — always pass both.
- ✅ `gapi drive search` CLI wrapper does NOT work for Shared Drives — always use direct REST.
- ✅ To get Shared Drive ID: it's the last path segment of the Drive URL, e.g. `drive.google.com/drive/folders/{{SHARED_DRIVE_ID}}` → ID = `{{SHARED_DRIVE_ID}}`
- ✅ To list Shared Drives available to the account: `GET /drive/v3/drives`

#### Shared Drive restructuring workflow (confirmed pattern — BusyCow Drive 2026-05-20)

1. **Scan first** — `list_folder(DRIVE_ID)` for root, then recurse per folder. Save all IDs to `/tmp/drive_id_map.json`.
2. **Create new folders** — `create_folder(name, parent_id)`. Save new IDs to `/tmp/drive_new_ids.json`.
3. **Rename root folders** — `rename(file_id, new_name)` before moving children to avoid confusion.
4. **Move in order** — move projects first (they reference other folders), then templates, then leaf files.
5. **`time.sleep(0.3)`** between move calls to avoid rate limit errors.
6. **Verify** — re-run `list_folder` tree after moves to confirm structure.

**BusyCow Drive naming convention (approved 2026-05-20):**
- Root level: `[DX] Folder` prefix for company-wide folders (floats to top, signals "shared by all BLs")
- Root level: bare name for Business Line folders (`[Product]`, `BusyCow`, `[Product]`, `[Product]`)
- Inside each BL: `00_Core`, `01_Sales & Marketing`, `02_Commercial`, `03_Clients`, `90_Inbox`, `Archived`
- `[DX] Projects/` at root — all time-bound projects regardless of BL, named `[BL] Project Name YYYY`
- `[DX] Operations/` — Invoices, Quotations, Templates, Contracts (cross-BL ops)
- `[DX] Company/` — company-level pitch, legal, brand (formerly empty `BusyCow/` folder)

#### List files in a folder (by folder ID from URL)

Use `--raw-query` with the `parents in` syntax. The folder ID is the last segment of the Google Drive URL.

```bash
FOLDER_ID="1U7KwezW-igc45KkNTnSKxRwnZfmaEQgA"
$GAPI drive search "'$FOLDER_ID' in parents" --raw-query --max 50
```

PITFALL: `gws_bridge.py drive files list --params '{"q": "..."}' ` does NOT work for folder queries — returns empty or 400 Invalid Value. Always use the `$GAPI drive search ... --raw-query` wrapper for folder listing.

#### Download a .docx / binary file from Drive

Files stored as `.docx` (not native Google Docs) CANNOT be exported via `files export` — that API only works for native Docs/Sheets/Slides and returns error: `fileNotExportable`. Use `files get` with `alt=media` instead:

```python
import json, os
from hermes_tools import terminal

hermes_home = os.environ.get("HERMES_HOME", os.path.expanduser("~/.hermes"))
skill_dir = f"{hermes_home}/skills/productivity/google-workspace"
python_bin = f"{hermes_home}/hermes-agent/venv/bin/python"
gbridge = f"{python_bin} {skill_dir}/scripts/gws_bridge.py"

file_id = "FILE_ID_HERE"
params = json.dumps({"fileId": file_id, "alt": "media"})
# IMPORTANT: output path must be RELATIVE — gws rejects absolute paths with a validation error
terminal(f"cd /tmp && {gbridge} drive files get --params '{params}' -o myfile.docx")
```

PITFALL: If you pass an absolute path like `-o /tmp/myfile.docx`, gws rejects it with: `--output resolves to ... which is outside the current directory`. Always `cd` to the target dir first and use a relative filename.

#### Read a Google Doc shared via public link (no auth required)

When a Google Doc is shared as "Anyone with the link can view", you can export it directly via curl **without OAuth** — the export URL is public:

```bash
DOC_ID="1afrXCOjR_D5itEGeTuei_wMEDWjdtX68"
curl -s -L -o /tmp/doc.txt "https://docs.google.com/document/d/${DOC_ID}/export?format=txt"
# Other formats: format=docx, format=pdf, format=html
```

PITFALL: `browser_navigate` to the export URL triggers a download and returns an error (`Download is starting`) — use curl instead.

PITFALL: The Google Docs API (`gws docs get DOC_ID`) requires the Docs API to be enabled in Google Cloud Console. If you get a 403 "Google Docs API has not been used", use the curl export approach above for public docs, or enable the API at https://console.developers.google.com/apis/api/docs.googleapis.com/overview.

PITFALL: **Google Sheets stored as .xlsx (Office format) cannot be read via the Sheets API** — the API returns `400: This operation is not supported for this document. The document must not be an Office file.` when calling `spreadsheets.get` or `spreadsheets.values.get`. These files show `rtpof=true` in their share URL. To read them: download via `drive files get --params '{"fileId": "ID", "alt": "media"}'` then parse with openpyxl. If the file returns 404, it's not in the authorized user's Drive (shared-only files are invisible to the API — user must add to their Drive first).

PITFALL: `drive files export` only works for **native Google Docs** (Docs/Sheets/Slides created inside Google Drive). If the file is a `.docx` upload, it returns `fileNotExportable`. Use `drive files get --params '{"fileId": "...", "alt": "media"}'` for uploaded binary files.

**PITFALL: `drive files get` (metadata or media) returns 404 if the file was shared with you via a public link but is not in your Drive. The Drive API only sees files the authenticated user owns or has been explicitly added to their Drive.

PITFALL: **"Anyone with the link" sharing does NOT grant API access.** A Google Doc shared as "Anyone with the link can view/edit" still returns 404 via the Drive API and a login-redirect HTML page via curl export if the OAuth account is not explicitly listed as a collaborator by email. Fix: owner must open Share → add the OAuth account (e.g. {{GOOGLE_OAUTH_EMAIL}}) by name. Direct file shares (Drive attachments shared to the email) work immediately — the "Anyone with link" gap only affects Docs/files not explicitly added. Google Workspace org restrictions may also block external sharing at the admin level regardless of share settings.

 A Google Doc shared via "Anyone with the link can view/edit" still returns 404 via the Drive API if the OAuth account is not explicitly listed as a collaborator (editor/viewer by email). The export URL (`/export?format=txt`) via curl also fails with a login redirect page. Fix: the owner must open Share → add the OAuth account email (e.g. {{GOOGLE_OAUTH_EMAIL}}) explicitly by name. Files shared directly to the email (like Drive file attachments) work immediately. Google Docs in another user's Workspace org may also be restricted at the org level regardless of share settings.

PITFALL: **"Shared with {{GOOGLE_OAUTH_EMAIL}}" ≠ immediately API-accessible.** Even after the user shares a doc, the Drive API can return 404 or 403. Two root causes:
1. The share was set to "Anyone with the link" — the API requires the email be added **directly** as an editor/viewer (confirm in the share dialog that the email shows up by name, not just "Anyone").
2. The doc is inside a **restricted Google Workspace org** that blocks cross-org API access — even when the browser link works. In that case, the owning org's OAuth app is the only one that can access via API.
- 404 on metadata = file is invisible to the account entirely.
- 403 PERMISSION_DENIED from Docs API = file is in Drive but org policy blocks it.
- Workaround: ask user to confirm email is listed by name in share dialog; if still 404, offer to draft in a new doc under the accessible account instead.

**PITFALL: "Anyone with the link" sharing does NOT make a file accessible via Drive API.** If a doc is shared as "Anyone with the link can view/edit", the Drive API still returns 404 for accounts that haven't been directly granted access. The authenticated OAuth account (e.g. `{{GOOGLE_OAUTH_EMAIL}}`) must be **explicitly listed as an editor/viewer** in the Share dialog (by email, not just link sharing). Symptoms:
- Drive metadata: `404 File not found`
- Docs API: `403 PERMISSION_DENIED`
- These are two different errors but both mean the same thing: the file is not visible to the account
- Force-refreshing the token does NOT help — the issue is share permissions, not token freshness

**Diagnosis checklist when seeing 404/403 on a supposedly-shared doc:**
1. Confirm the share dialog shows the OAuth email listed **by name** under "People with access"
2. If it shows only "Anyone with the link" — the file owner must add the email directly
3. If the file is in a Google Workspace org (e.g. a company domain), external sharing may be blocked by the org's admin policy — the owner must check Workspace sharing settings
4. Token refresh will not fix a permissions issue — check permissions first before refreshing

PITFALL: **Image-based PDFs return 0 chars from pymupdf** — `page.get_text()` returns empty string, and `get_text("blocks")` returns only one block of type `1` (image). These PDFs are scans with no embedded text layer. Detection: `page.get_text("dict")["blocks"]` returns a single block with `"type": 1` and `"image"` key.

**OCR pipeline for image-based PDFs:**
```bash
# Install once
sudo apt-get install -y tesseract-ocr poppler-utils
~/.hermes/hermes-agent/venv/bin/pip3 install pytesseract pdf2image
```
```python
# Run as background job (OCR is slow — 1-3 min per PDF at dpi=150)
import pytesseract
from pdf2image import convert_from_path

images = convert_from_path("/tmp/file.pdf", dpi=150)  # dpi=150 is good tradeoff speed/quality
full_text = ""
for i, img in enumerate(images):
    full_text += f"\n--- Page {i+1} ---\n{pytesseract.image_to_string(img)}"
with open("/tmp/file_ocr.txt", "w") as f:
    f.write(full_text)
```
Always run OCR as `terminal(background=True, notify_on_complete=True)` — it takes 2-5 min for a 10-page PDF. Never run synchronously.

PITFALL: `web_extract("file:///tmp/...")` is **blocked** — the tool rejects local file:// URLs as "private network address". To read downloaded PDFs, use `pymupdf` via terminal instead:
```python
from hermes_tools import terminal
r = terminal("""
source ~/.hermes/hermes-agent/venv/bin/activate && python3 -c "
import pymupdf
doc = pymupdf.open('/tmp/file.pdf')
for i, page in enumerate(doc):
    print(f'--- Page {i+1} ---')
    print(page.get_text())
"
""")
print(r['output'])
```
`pymupdf` is available in the Hermes venv. Works for all PDFs including scanned ones (with embedded text).

**PITFALL: `drive files copy` returns 404 if the template file is not in the authenticated user's Drive — even if it's publicly shared. The file must be owned by or explicitly shared (edit/view) with the OAuth account. Ask the user to share the file with the OAuth account's email, or use `Make a copy` in Google Docs UI first.

**PITFALL: Cannot move My Drive files directly into a Shared Drive** — `files.update` with `addParents=<shared_drive_folder>` returns 403 even with `supportsAllDrives=true` if the file is owned by another Google account and `canMoveItemIntoTeamDrive=false`. Workaround: use `files.copy` to copy the file into the Shared Drive folder instead. The original stays in My Drive (user can delete manually). Pattern:
```python
result = gapi("POST", f"files/{doc_id}/copy",
    {"name": name, "parents": [TARGET_SHARED_FOLDER_ID]},
    {"supportsAllDrives": "true"}
)
new_id = result["id"]
```
Check `capabilities.canMoveItemIntoTeamDrive` on the file metadata first — if False, go straight to copy.

PITFALL: Google Docs API (`docs batchUpdate`) requires the **`documents`** scope (not `documents.readonly`) and the **Docs API must be enabled** in Google Cloud Console. If 403: enable at https://console.developers.google.com/apis/api/docs.googleapis.com/overview?project=PROJECT_ID

PITFALL: OAuth scopes for full Invoice workflow require `drive` (not `drive.readonly`) and `documents` (not `documents.readonly`). Update setup.py SCOPES list and re-run `--auth-url` + `--auth-code` flow to get new token with write scopes.

#### Extract text from downloaded .docx files

`python-docx` (`import docx`) is available in the Hermes venv:

```python
import docx

doc = docx.Document("/tmp/myfile.docx")
text = "\n".join(p.text for p in doc.paragraphs if p.text.strip())
print(text)
```

### Contacts

```bash
$GAPI contacts list --max 20
```

### Sheets

```bash
$GAPI sheets get SHEET_ID "Sheet1!A1:D10"
$GAPI sheets update SHEET_ID "Sheet1!A1:B2" --values '[["Name","Score"],["Alice","95"]]'
$GAPI sheets append SHEET_ID "Sheet1!A:C" --values '[["new","row","data"]]'
```

### Docs

```bash
$GAPI docs get DOC_ID
```

### Patching Calendar Events (attendees, etc.)

**PITFALL: `gws calendar events patch` does NOT correctly handle array fields like `attendees`** — it stringifies the array instead of sending it as JSON. Use the Google Calendar REST API directly via Python instead:

```python
import json, os, sys, urllib.request

hermes_home = os.environ.get("HERMES_HOME", os.path.expanduser("~/.hermes"))
skill_dir = f"{hermes_home}/skills/productivity/google-workspace"
sys.path.insert(0, f"{skill_dir}/scripts")
from gws_bridge import get_valid_token

access_token = get_valid_token()
headers = {"Authorization": f"Bearer {access_token}", "Content-Type": "application/json"}

EVENT_ID = "your_event_id"
body = {
    "attendees": [
        {"email": "alice@example.com"},
        {"email": "bob@example.com"}
    ]
}
data = json.dumps(body).encode()
req = urllib.request.Request(
    f"https://www.googleapis.com/calendar/v3/calendars/primary/events/{EVENT_ID}?sendUpdates=all",
    data=data, headers=headers, method="PATCH"
)
with urllib.request.urlopen(req, timeout=15) as resp:
    result = json.loads(resp.read())
    print(result.get("attendees"))
```

Pass `?sendUpdates=all` to notify attendees of the change.

### Direct gws access (advanced)

For operations not covered by the wrapper, use `gws_bridge.py` directly:

```bash
GBRIDGE="$PYTHON_BIN $GWORKSPACE_SKILL_DIR/scripts/gws_bridge.py"
$GBRIDGE calendar +agenda --today --format table
$GBRIDGE gmail +triage --labels --format json
$GBRIDGE drive +upload ./report.pdf
$GBRIDGE sheets +read --spreadsheet SHEET_ID --range "Sheet1!A1:D10"
```

## Output Format

All commands return JSON via `gws --format json`. Key output shapes:

- **Gmail search/triage**: Array of message summaries (sender, subject, date, snippet)
- **Gmail get/read**: Message object with headers and body text
- **Gmail send/reply**: Confirmation with message ID
- **Calendar list/agenda**: Array of event objects (summary, start, end, location)
- **Calendar create**: Confirmation with event ID and htmlLink
- **Drive search**: Array of file objects (id, name, mimeType, webViewLink)
- **Sheets get/read**: 2D array of cell values
- **Docs get**: Full document JSON (use `body.content` for text extraction)
- **Contacts list**: Array of person objects with names, emails, phones

Parse output with `jq` or read JSON directly.

### Working with Google Docs Tabs (Multi-Tab Documents)

Google Docs supports multiple tabs per document (visible in the UI as separate tabbed sections). The API requires explicit opt-in and tab targeting.

**Read a specific tab:**
```python
# MUST pass includeTabsContent=true — default response returns tabs=[] (empty)
doc = gapi("GET", f"https://docs.googleapis.com/v1/documents/{doc_id}",
           params={"includeTabsContent": "true"})

for tab in doc.get("tabs", []):
    tp = tab.get("tabProperties", {})
    print(tp.get("title"), tp.get("tabId"), tp.get("index"))
    # Body lives inside documentTab, not at top level
    body = tab.get("documentTab", {}).get("body", {}).get("content", [])
```

**Write to a specific tab:**
```python
# All location/range objects need a "tabId" field to target the right tab
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [{
        "insertText": {
            "location": {"index": 1, "tabId": "t.8zhabhu6lm95"},
            "text": "content here"
        }
    }]
})
```

**Apply styles to a specific tab:**
```python
style_requests.append({"updateParagraphStyle": {
    "range": {
        "startIndex": p["start"],
        "endIndex": p["end"] - 1,
        "tabId": "t.8zhabhu6lm95"   # ← required
    },
    "paragraphStyle": {"namedStyleType": "HEADING_1"},
    "fields": "namedStyleType"
}})
```

**Delete content in a specific tab:**
```python
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [{"deleteContentRange": {
        "range": {
            "startIndex": 1,
            "endIndex": end_index - 1,
            "tabId": "t.8zhabhu6lm95"
        }
    }}]
})
```

**PITFALL: `tabs=[]` without `includeTabsContent=true`** — the default GET response returns an empty `tabs` array. Always pass `?includeTabsContent=true` when working with tabbed docs. The tab ID is the fragment after `#tab=` in the document URL (e.g. `t.8zhabhu6lm95`).

**PITFALL: body content is nested deeper** — with tabs, content lives at `tab["documentTab"]["body"]["content"]`, not `doc["body"]["content"]`. The top-level `doc["body"]` is empty when tabs are present.

---

### Creating a Long-Form Google Doc from Scratch (with Heading Styles)

Use this pattern when generating a structured document (report, spec, proposal) programmatically — not from a template.

### The Two-Pass Approach (confirmed working — 2026-05-21)

**Why two passes:** You must insert all text first, then read the document back to get paragraph `startIndex`/`endIndex` values, then apply styles. You cannot predict indices before insertion because Google Docs assigns them after the API inserts content.

**Pass 1 — Create doc and insert all text:**
```python
import sys, os, json, urllib.request

hermes_home = os.environ.get("HERMES_HOME", os.path.expanduser("~/.hermes"))
skill_dir = f"{hermes_home}/skills/productivity/google-workspace"
sys.path.insert(0, f"{skill_dir}/scripts")
from gws_bridge import get_valid_token

token = get_valid_token()
headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}

def gapi(method, url, body=None):
    data = json.dumps(body).encode() if body else None
    req = urllib.request.Request(url, data=data, headers=headers, method=method)
    with urllib.request.urlopen(req, timeout=30) as r:
        return json.loads(r.read())

# 1. Create the doc
result = gapi("POST", "https://docs.googleapis.com/v1/documents", {"title": "My Document Title"})
doc_id = result["documentId"]

# 2. Insert ALL text in one batchUpdate at index 1
doc_text = "Title Line\nSection 1 Heading\nBody paragraph...\n"
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [{"insertText": {"location": {"index": 1}, "text": doc_text}}]
})
```

**Pass 2 — Read paragraph indices, then apply styles:**
```python
# 3. Read back the doc to get paragraph positions
doc = gapi("GET", f"https://docs.googleapis.com/v1/documents/{doc_id}")

paragraphs = []
for elem in doc["body"]["content"]:
    if "paragraph" in elem:
        text = ""
        for pe in elem["paragraph"].get("elements", []):
            if "textRun" in pe:
                text += pe["textRun"].get("content", "")
        paragraphs.append({
            "start": elem["startIndex"],
            "end": elem["endIndex"],
            "text": text.strip()
        })

# 4. Find headings by regex pattern, collect (start, end) tuples
import re
h1_ranges = []
h2_ranges = []
for p in paragraphs:
    if re.match(r'^\d+\.\s+[A-Z]', p["text"]) and len(p["text"]) < 80:
        h1_ranges.append((p["start"], p["end"] - 1))
    elif re.match(r'^\d+\.\d+\s+[A-Z]', p["text"]) and len(p["text"]) < 80:
        h2_ranges.append((p["start"], p["end"] - 1))

# 5. Apply styles in one batchUpdate
style_requests = []

# Title
style_requests.append({
    "updateParagraphStyle": {
        "range": {"startIndex": paragraphs[0]["start"], "endIndex": paragraphs[0]["end"] - 1},
        "paragraphStyle": {"namedStyleType": "TITLE"},
        "fields": "namedStyleType"
    }
})

for s, e in h1_ranges:
    style_requests.append({
        "updateParagraphStyle": {
            "range": {"startIndex": s, "endIndex": e},
            "paragraphStyle": {"namedStyleType": "HEADING_1"},
            "fields": "namedStyleType"
        }
    })

for s, e in h2_ranges:
    style_requests.append({
        "updateParagraphStyle": {
            "range": {"startIndex": s, "endIndex": e},
            "paragraphStyle": {"namedStyleType": "HEADING_2"},
            "fields": "namedStyleType"
        }
    })

gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate",
     {"requests": style_requests})
```

### Named Style Types
| Style | `namedStyleType` value |
|---|---|
| Document title | `TITLE` |
| Subtitle | `SUBTITLE` |
| H1 | `HEADING_1` |
| H2 | `HEADING_2` |
| H3 | `HEADING_3` |
| Body | `NORMAL_TEXT` |

### Targeted Slide Block Replacement (find-by-header pattern)

When rewriting one or two slides in a multi-slide doc without touching the rest, use paragraph index scanning to find the exact range:

```python
doc = gapi("GET", f"https://docs.googleapis.com/v1/documents/{doc_id}")
paragraphs = []
for elem in doc["body"]["content"]:
    if "paragraph" in elem:
        text = "".join(
            pe["textRun"].get("content", "")
            for pe in elem["paragraph"].get("elements", [])
            if "textRun" in pe
        )
        paragraphs.append({"start": elem["startIndex"], "end": elem["endIndex"], "text": text.strip()})

# Find target slide start and next slide start
target_start = next_start = None
for p in paragraphs:
    if "Slide N｜Target Title" in p["text"] and target_start is None:
        target_start = p["start"]
    if target_start and "Slide N+1｜" in p["text"]:
        next_start = p["start"]
        break

# Delete old block, insert new one in one batchUpdate
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [
        {"deleteContentRange": {"range": {"startIndex": target_start, "endIndex": next_start - 1}}},
        {"insertText": {"location": {"index": target_start}, "text": new_slide_text}}
    ]
})
```

**Key points:**
- Always re-read paragraph indices AFTER any prior edit — indices shift
- Use a placeholder trick when `replaceAllText` won't match a section header: replace a unique short string first (1 hit confirmed), then re-read indices and do delete+insert
- `next_start - 1` as the delete end excludes the newline before the next slide header
- If replacing Slides 3 AND 4, replace S3 first, re-read doc, then find S4's new position before replacing it

### Full Doc Rewrite Pattern (clear + reinsert)

When a doc needs to be **completely replaced** (not edited), use this sequence:

```python
# 1. Get current end_index
doc = gapi("GET", f"https://docs.googleapis.com/v1/documents/{doc_id}")
end_index = doc["body"]["content"][-1]["endIndex"]

# 2. Delete all content except last char (index 1 to end_index - 1)
if end_index > 2:
    gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
        "requests": [{"deleteContentRange": {"range": {"startIndex": 1, "endIndex": end_index - 1}}}]
    })

# 3. Insert new content at index 1
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [{"insertText": {"location": {"index": 1}, "text": new_doc_text}}]
})

# 4. Re-read doc and apply styles (two-pass pattern)
```

This is the correct pattern for iterative document drafting — reuse the same doc_id across versions rather than creating new docs each time.

### Iterative Patching Pattern for Multi-Version Documents

When a long-form Google Doc needs multiple rounds of content updates (e.g. investor docs revised across a session), use **targeted `replaceAllText` batches** rather than delete-all + reinsert. This preserves heading styles and avoids re-styling the entire doc from scratch.

**Pattern: replaceAllText batch**
```python
def r(old, new):
    return {"replaceAllText": {
        "containsText": {"text": old, "matchCase": True},
        "replaceText": new
    }}

requests = [r("old text 1", "new text 1"), r("old text 2", "new text 2")]
result = gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate",
              {"requests": requests})
# Check hit count per replacement:
for i, reply in enumerate(result.get("replies", [])):
    n = reply.get("replaceAllText", {}).get("occurrencesChanged", 0)
    print(f"Patch {i}: {n} hit(s)")
```

**PITFALL: `replaceAllText` on multi-line strings almost always returns 0 hits** — even when the string looks correct. Two causes compound: (1) Google Docs exports with `\r\n` line endings, not `\n`; (2) empty paragraphs between blocks add extra `\r\n\r\n` that your match string doesn't include. Diagnosis: export the doc as `text/plain` via Drive API and use `repr()` to inspect the exact byte sequence around your target string before building the match.

When `replaceAllText` fails on a block you need to replace, fall back to the **paragraph-index delete+reinsert pattern**:
```python
# 1. Read doc to get paragraph index ranges
doc = gapi("GET", f"https://docs.googleapis.com/v1/documents/{doc_id}")
paragraphs = []
for elem in doc["body"]["content"]:
    if "paragraph" in elem:
        text = "".join(pe["textRun"].get("content","") for pe in elem["paragraph"].get("elements",[]) if "textRun" in pe)
        paragraphs.append({"start": elem["startIndex"], "end": elem["endIndex"], "text": text.strip()})

# 2. Identify the start/end indices of the block you want to replace
delete_start = paragraphs[N]["start"]   # first para to delete
delete_end   = paragraphs[M]["end"] - 1 # last para to delete (exclusive of its final \n)

# 3. Delete then insert in one batchUpdate — order matters (delete first)
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [
        {"deleteContentRange": {"range": {"startIndex": delete_start, "endIndex": delete_end}}},
        {"insertText": {"location": {"index": delete_start}, "text": new_text}}
    ]
})
```
This is reliable regardless of line ending quirks. Use it whenever `replaceAllText` returns 0 hits after two attempts. Two fixes:
1. Use short unique single-line anchors (most reliable — avoids the problem entirely)
2. Export the doc and check exact line endings with `repr()` before building the match string

```python
# Diagnose before matching multi-line blocks:
req = urllib.request.Request(
    f"https://www.googleapis.com/drive/v3/files/{doc_id}/export?mimeType=text/plain",
    headers=headers
)
with urllib.request.urlopen(req, timeout=30) as r:
    text = r.read().decode("utf-8")
idx = text.find("Section heading you want to match")
print(repr(text[idx:idx+300]))  # see exact line endings
```

**PITFALL: 0 hits on multi-line match** — Split into multiple single-sentence patches rather than one large block. Each sentence is unique enough to match reliably.

**Structural audit before final delivery** — Before sending to users, export and scan for:
1. Duplicate content (appendices that repeat earlier sections — cut them)
2. Broken section numbering (new sections inserted without renumbering children)
3. Two tables with conflicting numbers on the same topic (investor will notice)
4. Orphaned blocks with no heading (content floating between sections)
5. Low-value redundant sections (sub-sections that restate parent section conclusion)

**Full delete + reinsert** is better when: content is >50% changed, or heading structure changes significantly. After full reinsert, always re-run the heading style pass:
```python
# Re-apply styles after full reinsert
import re
for p in paragraphs[2:]:
    t = p["text"]
    if re.match(r'^\d+\.\s+[A-Z][A-Z\s\-—&,:\/]+$', t) and len(t) < 100:
        # → HEADING_1
    elif re.match(r'^\d+\.\d+[a-z]?\s+', t) and len(t) < 120:
        # → HEADING_2
    elif re.match(r'^APPENDIX [A-Z]', t):
        # → HEADING_1
```

### Pitfalls for Long-Form Doc Creation

- ❌ **Do NOT try to calculate indices before insertion** — Google Docs assigns indices after content is written. Always read the doc back after insertion.
- ❌ **`endIndex` includes the newline character** — when setting a style range, use `endIndex - 1` to avoid styling the newline, which can bleed the style into the next paragraph.
- ✅ **Insert all text in a single `insertText` request at index 1** — multiple sequential inserts at index 1 reverse the order. If you must insert in chunks, insert in reverse order, or concatenate into one string.
- ✅ **TOC vs content duplicates**: If your text includes a Table of Contents listing headings before the actual sections, the regex will match both. Filter by `startIndex > <threshold>` to only style the actual content sections, not the TOC entries.
- ✅ **Batch all style requests into one `batchUpdate`** — fewer API calls, and avoids index drift between calls.
- ✅ **`HEADING_3` is good for named sub-sections** (e.g. `COMPONENT A —`, `PHASE 1 —`) identified by regex `r'^(COMPONENT|PHASE|CONTROL|SHIFT|PRINCIPLE)'`.

## Invoice Generation via Google Docs Template

Pattern confirmed working (2026-05-15):

1. **Template must be in the authorized user's My Drive** — shared-only files return 404 on copy even with full Drive scope. User must "Make a copy" or share the file with the authorized Google account.
2. **Copy template**: `drive files copy --params '{"fileId": "TEMPLATE_ID", "name": "New Name"}'`
3. **Fill placeholders**: `docs batchUpdate` with `replaceAllText` requests
4. **Export PDF**: `drive files get export?mimeType=application/pdf`
5. **Enable Google Docs API** in Google Cloud Console before first use — `403 Google Docs API has not been used` otherwise

```python
# Full working pattern
access_token = refresh_google_token()
H = {'Authorization': f'Bearer {access_token}', 'Content-Type': 'application/json'}

# 1. Copy
r = gapi('POST', f'https://www.googleapis.com/drive/v3/files/{TEMPLATE_ID}/copy',
    {'name': 'Invoice Name'})
doc_id = r['id']

# 2. Fill
requests = [{'replaceAllText': {
    'containsText': {'text': k, 'matchCase': True}, 'replaceText': v
}} for k, v in replacements.items()]
gapi('POST', f'https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate',
    {'requests': requests})

# 3. Export PDF
req = urllib.request.Request(
    f'https://www.googleapis.com/drive/v3/files/{doc_id}/export?mimeType=application/pdf',
    headers={'Authorization': f'Bearer {access_token}'})
with urllib.request.urlopen(req, timeout=30) as r:
    pdf_data = r.read()
```

**Scopes required**: `drive` (NOT `drive.readonly`) + `documents` (NOT `documents.readonly`)

## Patching Calendar Events (attendees, location, etc.)

The `gws calendar events patch` CLI **cannot handle array fields** — passing `attendees` as a repeated param stringifies it instead of sending a JSON array, leaving the original attendees unchanged.

Use the Google Calendar REST API directly for any PATCH that involves arrays:

```python
import json, os, sys, urllib.request

hermes_home = os.environ.get("HERMES_HOME", os.path.expanduser("~/.hermes"))
skill_dir = f"{hermes_home}/skills/productivity/google-workspace"
sys.path.insert(0, f"{skill_dir}/scripts")
from gws_bridge import get_valid_token

access_token = get_valid_token()
headers = {"Authorization": f"Bearer {access_token}", "Content-Type": "application/json"}

EVENT_ID = "YOUR_EVENT_ID"
body = {
    "attendees": [
        {"email": "alice@example.com"},
        {"email": "bob@example.com"}
    ]
}
data = json.dumps(body).encode()
req = urllib.request.Request(
    f"https://www.googleapis.com/calendar/v3/calendars/primary/events/{EVENT_ID}?sendUpdates=all",
    data=data, headers=headers, method="PATCH"
)
with urllib.request.urlopen(req, timeout=15) as resp:
    result = json.loads(resp.read())
    print([a["email"] for a in result.get("attendees", [])])
```

`sendUpdates=all` ensures all attendees receive the updated invite notification.

The `get_valid_token()` function (not `refresh_google_token`) is the correct import from `gws_bridge.py`.

## Taking a Screenshot of a Local HTML File (Puppeteer)

Use when user asks for a screenshot of an HTML artifact (infographic, diagram, prototype) produced by the agent.

**Confirmed working pattern (2026-05-22):**

```bash
# Step 1: Install Chrome (one-time, only if missing)
~/.hermes/node/bin/puppeteer browsers install chrome
# Installs to ~/.cache/puppeteer/chrome/linux-<version>/chrome-linux64/chrome

# Step 2: Write screenshot script
cat > /tmp/screenshot.mjs << 'EOF'
import { createRequire } from 'module';
const require = createRequire(import.meta.url);
const puppeteer = require('~/.hermes/node/lib/node_modules/puppeteer');

const browser = await puppeteer.launch({
  args: ['--no-sandbox', '--disable-setuid-sandbox', '--disable-dev-shm-usage'],
  headless: true
});
const page = await browser.newPage();
await page.setViewport({ width: 1440, height: 900, deviceScaleFactor: 2 });
await page.goto('file:///tmp/your_file.html', { waitUntil: 'networkidle0' });
await page.screenshot({ path: '/tmp/output.png', fullPage: true });
await browser.close();
console.log('OK');
EOF

# Step 3: Run
node /tmp/screenshot.mjs
```

**Key pitfalls:**
- ❌ `import puppeteer from 'puppeteer'` → `ERR_MODULE_NOT_FOUND` — puppeteer is installed globally at `~/.hermes/node/lib/node_modules/puppeteer`, NOT in node's default resolution path
- ✅ Use `createRequire(import.meta.url)` + absolute path to require it
- ❌ `--headless=new` flag not needed — default `headless: true` works
- ❌ `execute_code` sandbox cannot run puppeteer — must use `terminal()`
- ✅ After screenshot, display with `vision_analyze('/tmp/output.png')` to verify before sending
- ✅ Use `MEDIA:/tmp/output.png` in response to deliver the image to Feishu/Telegram

**Full delivery pattern:**
```python
# 1. Take screenshot via terminal()
# 2. Verify with vision_analyze()
# 3. Deliver with MEDIA:/tmp/output.png in response message
```

## Writing to a Specific Tab in a Multi-Tab Google Doc

When a doc has multiple tabs (e.g. "Slides Framework" and "Pitching"), you must pass `tabId` on every insert/delete/style operation, otherwise the API defaults to the first tab.

```python
# Get tab IDs
doc = gapi("GET", f"https://docs.googleapis.com/v1/documents/{doc_id}",
           params={"includeTabsContent": "true"})
for tab in doc.get("tabs", []):
    tp = tab.get("tabProperties", {})
    print(tp["title"], tp["tabId"])  # e.g. "Pitching" → "t.8zhabhu6lm95"

# Insert text into a specific tab
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [{
        "insertText": {
            "location": {"index": 1, "tabId": "t.8zhabhu6lm95"},
            "text": "..."
        }
    }]
})

# Delete content in a specific tab (clear before rewrite)
gapi("POST", f"https://docs.googleapis.com/v1/documents/{doc_id}:batchUpdate", {
    "requests": [{
        "deleteContentRange": {
            "range": {"startIndex": 1, "endIndex": end_index - 1, "tabId": "t.8zhabhu6lm95"}
        }
    }]
})

# Apply styles in a specific tab
style_requests.append({"updateParagraphStyle": {
    "range": {"startIndex": p["start"], "endIndex": p["end"]-1, "tabId": "t.8zhabhu6lm95"},
    "paragraphStyle": {"namedStyleType": "HEADING_1"},
    "fields": "namedStyleType"
}})
```

**⚠️ `doc.get("tabs", [])` returns empty unless you pass `includeTabsContent=true`** in the GET request. Without it, tabs appears as an empty list even when multiple tabs exist.

## Finding Existing Docs Before Creating New Ones

Before creating a new Google Doc, **always search first** — there may already be a relevant doc in Drive that should be updated instead. Search across both My Drive and Shared Drive:

```python
params = {
    "includeItemsFromAllDrives": "true",
    "supportsAllDrives": "true",
    "q": "(name contains 'keyword' or fullText contains 'keyword') and mimeType='application/vnd.google-apps.document'",
    "fields": "files(id,name,webViewLink,modifiedTime)",
    "pageSize": "20",
    "orderBy": "modifiedTime desc"
}
files = gapi("GET", "files", params_dict=params).get("files", [])
```

**PITFALL:** Searching by `fullText contains` in Shared Drive requires `corpora=drive` + `driveId`. Without it, `fullText` search may return empty even if the file exists.

## Moving a Doc/File into a Folder

To move an existing file into a new folder, you must pass BOTH `addParents` and `removeParents` to the PATCH endpoint:

```python
# First, get current parents
doc_meta = gapi("GET", f"files/{file_id}", params_dict={"supportsAllDrives": "true", "fields": "id,name,parents"})
current_parents = doc_meta.get("parents", [])

# Then move
gapi("PATCH", f"files/{file_id}",
    {},
    {
        "addParents": new_folder_id,
        "removeParents": ",".join(current_parents),
        "supportsAllDrives": "true",
        "fields": "id,name,parents"
    })
```

**PITFALL:** Omitting `removeParents` leaves the file in both the old and new location — it does NOT move, it copies the parent reference.

## Standard Post-Doc-Creation Routine

Every time a Google Doc is created for Hunter/BusyCow, always do all three steps in order:

1. **Share edit rights** to `{{OWNER_EMAIL}}` and `{{ADMIN_EMAIL}}`:
```python
emails = ["{{OWNER_EMAIL}}", "{{ADMIN_EMAIL}}"]
for email in emails:
    body = {"role": "writer", "type": "user", "emailAddress": email}
    data = json.dumps(body).encode()
    req = urllib.request.Request(
        f"https://www.googleapis.com/drive/v3/files/{doc_id}/permissions?sendNotificationEmail=false",
        data=data, headers=headers, method="POST"
    )
    with urllib.request.urlopen(req, timeout=15) as resp:
        print(json.loads(resp.read()))
```

2. **Send the link via Feishu** to `{{OWNER_TASK_CHAT_ID}}` (Hunter's Task Tracker chat):
```
📄 Google Doc created: <Title>
https://docs.google.com/document/d/<DOC_ID>/edit
```

3. **Confirm** in the chat response that both sharing and Feishu message are done.

---

## Rules

1. **Never send email or create/delete events without confirming with the user first.**
2. **"Calendar invite" always means Google Calendar** — even if the team uses Feishu/Lark for messaging. Default to Google Calendar unless the user explicitly says "Feishu Calendar" or "Lark Calendar".
2. **Check auth before first use** — run `setup.py --check`.
3. **Use the Gmail search syntax reference** for complex queries.
4. **Calendar times must include timezone** — ISO 8601 with offset or UTC.
5. **Respect rate limits** — avoid rapid-fire sequential API calls.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| `NOT_AUTHENTICATED` | Run setup Steps 2-5 |
| `REFRESH_FAILED` | Token revoked — redo Steps 3-5 |
| `gws: command not found` | Install: `npm install -g @googleworkspace/cli` |
| `HttpError 403` | Missing scope — `$GSETUP --revoke` then redo Steps 3-5 |
| `HttpError 403: Access Not Configured` | Enable API in Google Cloud Console |
| `Request had insufficient authentication scopes` | Token has read-only scopes. Update `SCOPES` in `setup.py` (see below) then re-auth |
| Advanced Protection blocks auth | Admin must allowlist the OAuth client ID |

## ⚠️ Write Scopes — Default is Read-Only

The default setup script uses `drive.readonly` and `documents.readonly`. These are **insufficient** for:
- Copying a Google Doc (`drive files copy`)
- Creating new Docs
- Replacing text in a Doc (Docs API batchUpdate)
- Uploading files

To enable write access, update `SCOPES` in `setup.py`:

```python
# ❌ Read-only (default) — cannot copy/create/modify:
"https://www.googleapis.com/auth/drive.readonly",
"https://www.googleapis.com/auth/documents.readonly",

# ✅ Full access — required for Invoice/template workflows:
"https://www.googleapis.com/auth/drive",
"https://www.googleapis.com/auth/documents",
```

Then re-run auth:
```bash
$GSETUP --auth-url
# User clicks link, pastes back redirect URL
$GSETUP --auth-code "THE_REDIRECT_URL"
$GSETUP --check  # should show AUTHENTICATED
```

## Revoking Access

```bash
$GSETUP --revoke
```
