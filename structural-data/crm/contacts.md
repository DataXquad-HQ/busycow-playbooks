# Contacts

**Table ID:** `tblJanYGuxNbhonD`
**Total Fields:** 17

People within accounts. The human layer of the CRM — deals and partnerships are tracked through contacts.

## Fields

| Field Name | Type | UI Variant | Notes |
|---|---|---|---|
| Contact Name | text | plain | Primary identifier |
| Title | text | plain | Job title |
| Department | text | plain | |
| Email | text | email | |
| Phone | text | phone | |
| Whatsapp | text | plain | |
| LinkedIn | text | url | |
| Status | select | — | Hot / Warm / Cold |
| Decision Role | select | — | See options below |
| Source | select | — | How this contact was acquired |
| Country | select | — | Taiwan / Hong Kong / China / Malaysia / Japan |
| Last Contact Date | datetime | yyyy/MM/dd | |
| Notes | text | plain | Free-form context |
| Company | link → Accounts | bidirectional | The account this contact belongs to |
| Engaging Deals | link → Deals | bidirectional | Deals this contact is involved in |
| Engaging Partnerships | link → Partnerships | bidirectional | Partnerships this contact is involved in |
| Activities | link → Engagements | bidirectional | All interactions with this contact |

## Select Options

### Status
`Hot` · `Warm` · `Cold`

### Decision Role
`Decision Maker` · `Champion` · `Influencer` · `End User` · `Gatekeeper`

### Source
`Outbound - Maya` · `Inbound - Web` · `Referral` · `Event` · `Partner`

### Country
`Taiwan` · `Hong Kong` · `China` · `Malaysia` · `Japan`
