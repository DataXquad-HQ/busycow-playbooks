# Partnerships

**Table ID:** `tbll9YwwO9xaAkGi`
**Total Fields:** 13

Partner relationships and channel pipeline. Tracks resellers, system integrators, and other channel partners from first contact through active partnership.

## Fields

| Field Name | Type | UI Variant | Notes |
|---|---|---|---|
| Partnership Name | text | plain | Primary identifier |
| Partnership ID | text | plain | Unique reference code |
| Stage | select | — | Pipeline stage for partner onboarding |
| Business Line | multiselect | — | Which product lines this partnership covers |
| Description | text | plain | Partnership context and background |
| Current Status Summary | text | plain | Latest status note |
| Target Close Date | datetime | yyyy-MM-dd | Target date to reach Active stage |
| Account | link → Accounts | bidirectional | The partner company |
| Primary Contact | link → Contacts | bidirectional | Main contact at the partner |
| Other Contacts | link → Contacts | unidirectional | Additional contacts at the partner |
| Activities | link → Engagements | bidirectional | All interactions on this partnership |
| Contract | link → Contracts | bidirectional | |
| Tasks | link → Sales Tasks | bidirectional | |

## Select Options

### Stage
`Prospect` · `Qualifying` · `Agreement` · `Active` · `On Hold` · `Inactive`

### Business Line (multi)
`BusyCow` · `GeoKernel` · `AquaOptima` · `TRACI` · `Distify` · `DataXquad`
