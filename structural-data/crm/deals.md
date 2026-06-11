# Deals

**Table ID:** `tblWPBazVVJU3PW5`
**Total Fields:** 26

Sales opportunities. One record per active deal, tracked from Lead through to Won or Lost.

## Fields

| Field Name | Type | UI Variant | Notes |
|---|---|---|---|
| Deal Name | text | plain | Primary identifier |
| Deal ID | text | plain | Unique deal reference code |
| Stage | select | — | Pipeline stage |
| Business Line | select | — | Which product this deal is for |
| Priority | select | — | High / Medium / Low |
| Expected Value | number | plain | Deal value in TWD (or local currency) |
| Probability % | number | plain | Estimated win probability 0–100 |
| Expected Close Date | datetime | yyyy/MM/dd | |
| Next Follow-up Date | datetime | yyyy/MM/dd | |
| Last Update Date | datetime | yyyy-MM-dd HH:mm | Auto-maintained |
| Description | text | plain | Deal context and background |
| Current Status Summary | text | plain | Latest status note |
| Next Action Summary | text | plain | What needs to happen next |
| Health Check | select | — | Current deal health signal |
| Risk Indicator | select | — | Low / Medium / High |
| Week Review Status | select | — | Whether reviewed in weekly pipeline review |
| Owner | user (multi) | — | Deal owner(s) |
| Doc Link | text | url | Link to proposal / supporting doc |
| Client | link → Accounts | bidirectional | The account this deal belongs to |
| Primary Contact | link → Contacts | bidirectional | Main point of contact |
| Other Contacts | link → Contacts | unidirectional | Secondary contacts on this deal |
| Activities | link → Engagements | bidirectional | All interactions on this deal |
| Quotations | link → Quotations | bidirectional | |
| Invoices | link → Invoices | bidirectional | |
| Contract | link → Contracts | bidirectional | |
| Tasks | link → Sales Tasks | bidirectional | |

## Select Options

### Stage
`Lead` · `Qualified` · `Proposal` · `Negotiation` · `Won` · `Lost`

### Business Line
`BusyCow` · `GeoKernel` · `AquaOptima` · `TRACI` · `Distify` · `DataXquad`

### Priority
`High` · `Medium` · `Low`

### Health Check
`On Track` · `Awaiting Response` · `Needs Follow-up` · `At Risk`

### Risk Indicator
`Low` · `Medium` · `High`

### Week Review Status
`Reviewed` · `Pending` · `N/A`
