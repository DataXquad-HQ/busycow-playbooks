# Accounts

**Table ID:** `tbl8r3DKnN8R1Rk7`
**Total Fields:** 22

Companies — clients, prospects, partners, and vendors. The top-level entity in the CRM hierarchy.

## Fields

| Field Name | Type | UI Variant | Notes |
|---|---|---|---|
| Company Name | text | plain | Primary identifier |
| Registered Name (EN) | text | plain | Legal name in English |
| Registered Name (CH) | text | plain | Legal name in Chinese |
| Description | text | plain | Company overview |
| Enrichment Overview | text | plain | Web-enriched summary |
| Industry | select | — | See options below |
| Type | multiselect | — | Client / Partner / Prospect / Vendor / Direct |
| Status | select | — | Hot / Warm / Cold |
| Country | select | — | Taiwan / Hong Kong / Malaysia / Oman |
| Website | text | url | |
| Company Email | text | plain | |
| Company Phone | text | phone | |
| Company LinkedIn | text | url | |
| HQ Address | text | plain | |
| Last Contact Date | datetime | yyyy-MM-dd HH:mm | Derived from Activities |
| Last Enriched Date | datetime | yyyy-MM-dd HH:mm | Timestamp of last enrichment run |
| Contacts | link → Contacts | bidirectional | All contacts at this account |
| Opportunities | link → Deals | bidirectional | All deals for this account |
| Partnership | link → Partnerships | bidirectional | All partnerships with this account |
| Activities | link → Engagements | bidirectional | All interactions with this account |
| Invoices | link → Invoices | bidirectional | All invoices for this account |
| Contracts | link → Contracts | bidirectional | All contracts for this account |

## Select Options

### Industry
`Tech / SaaS` · `Healthcare` · `Manufacturing / Trading` · `Water / Utilities` · `Retail / E-commerce` · `Logistics / Transport` · `Construction / Property` · `Finance / Insurance` · `Education` · `Government / Public` · `F&B / Hospitality` · `Other`

### Type (multi)
`Client` · `Partner` · `Prospect` · `Vendor` · `Direct`

### Status
`Hot` · `Warm` · `Cold`

### Country
`Taiwan` · `Hong Kong` · `Malaysia` · `Oman`
