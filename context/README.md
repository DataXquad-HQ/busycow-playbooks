# Structural Data

Shared schema definitions that **multiple agents read and write**.
These are not owned by any single agent — they describe the data layer
the whole team operates on.

## Contents

| Path | What |
|------|------|
| `crm/SCHEMA.md` | Twenty CRM object definitions — standard + custom objects and fields |

## Conventions

- Schema files use `{{PLACEHOLDER}}` for any instance-specific IDs
- One `SCHEMA.md` per system/tool
- Do not embed credentials or app tokens here
- When an object or field is added to a live instance, update the SCHEMA.md here too
