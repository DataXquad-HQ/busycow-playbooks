# Structural Data

This directory documents the **live data schemas** of the DataXquad operational systems. Unlike playbooks (which describe what agents do), structural data documents what exists — the tables, fields, options, and relationships that agents read from and write to.

## Categories

| Directory | What it documents |
|---|---|
| [crm/](crm/) | Sales & Ops Lark Base — Accounts, Contacts, Deals, Partnerships, Engagements |

## Purpose

- **Source of truth** for CRM field names, types, and select options — referenced by skills and agents when writing records
- **Onboarding reference** for new agents or team members who need to understand the data model
- **Change log** — when schema changes are made in Lark, this directory should be updated to reflect them

## Update Policy

Schema changes in Lark Base should be reflected here within the same sprint. The agent responsible for a schema change (typically Leo or Iris) should open a PR updating the relevant file.
