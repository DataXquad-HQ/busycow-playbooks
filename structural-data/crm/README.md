# CRM Schema

Live schema of the DataXquad Sales & Ops Lark Base (`MtvNbgCHXaRAaUsWXsCjnekep2g`).

This directory documents the five core CRM tables. Each file describes the fields, types, select options, and cross-table relationships for one table.

## Tables

| Table | File | Purpose |
|---|---|---|
| Accounts | [accounts.md](accounts.md) | Companies — clients, prospects, partners, vendors |
| Contacts | [contacts.md](contacts.md) | People within accounts |
| Deals | [deals.md](deals.md) | Sales opportunities (pipeline) |
| Partnerships | [partnerships.md](partnerships.md) | Partner relationships and channel pipeline |
| Engagements | [engagements.md](engagements.md) | All interactions — calls, meetings, messages |

## Relationship Map

```
Accounts ──< Contacts
Accounts ──< Deals
Accounts ──< Partnerships
Contacts ──< Engagements
Deals    ──< Engagements
Partnerships ──< Engagements
Deals    ──< Contacts (Primary Contact + Other Contacts)
Partnerships ──< Contacts (Primary Contact + Other Contacts)
```

## Lark Base Token

`MtvNbgCHXaRAaUsWXsCjnekep2g`

## Field Type Reference

| Type | Description |
|---|---|
| `text` | Plain text, URL, phone, email variants |
| `number` | Numeric value |
| `select` | Single-select enum |
| `multiselect` | Multi-select enum |
| `datetime` | Date / datetime |
| `user` | Lark user reference |
| `link` | Bidirectional or unidirectional cross-table link |
