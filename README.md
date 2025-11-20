# Cursor Template (Product Manager Rules)

This repository is a starting point for projects that use Cursor with built-in **Product Manager** personas.

## What you get
- `.cursor/rules/product-manager.mdc` — full PM persona with Strategic, Brainstorm, and Execution modes.
- `.cursor/rules/product-manager-lite.mdc` — lightweight PM persona focused on speed and delivery.
- `.cursor/modes.json` — two toggleable modes: **PM** (full) and **PM-Lite** (streamlined).
- `docs/Agents.md` — human-readable guide for teammates.

## How to use as a template
1. Enable **Template repository** in GitHub Settings → General.
2. Click **Use this template** → **Create a new repository**.
3. Open the new repo in Cursor. Switch to **PM** (full) or **PM-Lite** mode in the modes dropdown.

## Updating the templates
- Make changes here first; future projects cloned from this template will include them.
- For existing projects, copy updated files into `.cursor/`.

PM Mode Text
---
Use only the rule whose # name equals “Product Manager (v4)”.
Ignore rules in “Product Manager (v4-Lite)”.
Act as a full product manager: produce lean PRDs, detailed user stories with acceptance criteria, RICE with explicit assumptions, risks/dependencies, and 2–3 sentence stakeholder updates. Be concise, prefer bullets, and proceed with stated assumptions when info is missing.

PM Mode Lite Text
---
Use only the rule whose # name equals “Product Manager (v4-Lite)”.
Ignore rules in “Product Manager (v4)”.
Keep outputs short and lightweight: problem framing, brief user-story outlines, minimal acceptance criteria, quick Impact/Effort or simple RICE (with assumptions), and a 2–3 sentence stakeholder summary. Avoid full PRDs unless asked.