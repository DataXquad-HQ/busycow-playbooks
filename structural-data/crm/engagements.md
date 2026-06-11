# Engagements

**Table ID:** `tblcpO0HxLr40wUZ`
**Total Fields:** 11

All interactions — calls, meetings, demos, messages. The activity log of the CRM. Every touchpoint with a contact or account is recorded here.

## Fields

| Field Name | Type | UI Variant | Notes |
|---|---|---|---|
| Title | text | plain | Short description of the interaction |
| Type | select | — | Channel / format of interaction |
| Date | datetime | yyyy/MM/dd | When the interaction took place |
| Status | select | — | Planned / Completed |
| Notes | text | plain | Full notes / summary from the interaction |
| Next Action | text | plain | What to do as a result of this engagement |
| Owner | user (multi) | — | Who conducted / owns this interaction |
| Contact | link → Contacts | bidirectional | The person interacted with |
| Account | link → Accounts | bidirectional | The company this engagement is under |
| Related Deal | link → Deals | bidirectional | The deal this engagement is tied to (optional) |
| Related Partnership | link → Partnerships | bidirectional | The partnership this engagement is tied to (optional) |

## Select Options

### Type
`Phone Call` · `In-person Visit` · `Online Meeting` · `WhatsApp / LINE` · `Demo` · `Message` · `Email` · `Event`

### Status
`Planned` · `Completed`

## Notes

- An Engagement does **not** require a linked Deal or Partnership. Nurture-stage interactions (e.g. check-ins with no active deal) are logged with only Account + Contact linked.
- `Next Action` is a free-text note for the BD rep — not a task. Tasks are created separately in Sales Tasks and linked to the relevant Deal or Partnership.
