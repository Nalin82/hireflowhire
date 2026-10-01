# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

HireFlow is a recruitment platform (job listings + job application flow). The repo is **pre-implementation**: there is no application code, build system, package manifest, or test suite yet. It currently contains:

- `design/`: design mockups and wireframes (the source of truth for UI)
- `.specify/` + `.claude/skills/speckit-*`: GitHub Spec Kit (v0.16.0) scaffolding for spec-driven development

When application code is added, update this file with its build/lint/test commands.

## Spec-driven workflow (Spec Kit)

Features are built through the Spec Kit cycle, invoked as skills:

`/speckit-constitution` → `/speckit-specify` → `/speckit-clarify` (optional) → `/speckit-plan` → `/speckit-tasks` → `/speckit-analyze` (optional) → `/speckit-implement`

- Helper scripts are **PowerShell** (`.specify/scripts/powershell/*.ps1`); the project was initialized with `script: ps`.
- Feature numbering is **sequential** (e.g. `001-...`, `002-...`).
- Templates for spec/plan/tasks/checklist artifacts live in `.specify/templates/`.
- `.specify/memory/constitution.md` is still the **unfilled template**. Run `/speckit-constitution` before planning, and fold the design rules below into it.

## Design files (`design/`)

- `*.dc.html` files are self-contained mockups rendered by `design/support.js`, a **generated** runtime (do not edit it). Each file wraps its markup in `<x-dc>` with an optional `<script type="text/x-dc" data-dc-script data-props=...>` block; `$preview.width` sets the intended canvas width.
- `ios-frame.jsx` is a device frame used by the mobile mockup.
- Key references: `HireFlow Style Sheet.dc.html` (colour, type, components), `Open Roles*.dc.html` (job listing), `Job Application*.dc.html` (application form, desktop + mobile).

## Brand and design rules (from `design/CLAUDE.md`, apply to all UI work)

- Primary blue `#1D4ED8` (hover `#1E40AF`), grey neutrals, Inter typeface, 8px spacing grid.
- Tone: warm and professional, not corporate-stiff.
- WCAG AA contrast everywhere.
- Mobile-first, must work down to 360px wide.
- Reuse a small set of simple components: button, text input, card.
- No gradients, no heavy drop shadows.
- **Standing instruction:** flag any request that conflicts with these rules before building it.
