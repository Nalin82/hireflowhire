# Implementation Plan: Job Application Form

**Branch**: `001-job-application-form` | **Date**: 2026-10-01 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/001-job-application-form/spec.md`

## Summary

A browser-only Angular app in `frontend/` rebuilds `design/Job Application Page.dc.html` (the 1a/2b
structure in the wireframes) for the Northgate Labs role. The form lives at `/apply` and the
confirmation at `/confirmation`. The form state is five-way and derived from signals. Saved
applications are appended to an `applications` array in localStorage through `ApplicationService`
and `StorageService`. The decisions are in [research.md](research.md).

## Technical Context

**Language/Version**: TypeScript (strict), Angular 22.2

**Primary Dependencies**: Angular (router, zoneless), Inter (Google Fonts). Dev only: angular-eslint,
Prettier.

**Storage**: browser `localStorage`, key `applications` ([schema](contracts/storage-schema.md))

**Testing**: Vitest via `ng test`, unit tests only (pure functions and both services)

**Target Platform**: evergreen desktop and mobile browsers, **no SSR (pinned, see R1)**

**Project Type**: single-page web app (front end only)

**Performance Goals**: interactions feel instant. The submitting state lasts at least 600ms.

**Constraints**: works from 360px wide; no network or back end; the CV file is never read or stored

**Scale/Scope**: 1 role, 2 routes, about 12 components, a few saved records per browser

## Constitution Check

*Gate: checked before Phase 0, and again after Phase 1.*

| Principle | Check | Status |
|---|---|---|
| I. Tokens | Tokens come from the style sheet. Page-only values are marked as page-sourced (R9). | ✅ |
| I. Fidelity | 1a/2b layout and breakpoints at 420 and 900px. The deviations are listed below. | ✅ (with deviations) |
| II. States | All five states are defined (R4). Validating and submitting are defined in the spec. | ✅ |
| III. Angular | Feature-first structure, lazy routes, standalone, OnPush, signal APIs, built-in control flow, root services, `hf-` prefix, suffix naming (R2) | ✅ |
| IV. Done | The acceptance criteria map to unit tests (contracts/services.md) and the manual checks M1–M12 (quickstart). There are no UI or e2e tests. | ✅ |
| V. Content | All copy comes from the design. The email-copy line is corrected (FR-019). | ✅ |
| Brand/a11y | AA colours, focus ring, labels, aria wiring (R8) | ✅ |

Post-design re-check: no new violations.

## Design Deviations (recorded per Principle I)

| Deviation | Reason |
|---|---|
| The confirmation is its own route instead of an in-place swap | Plan input. The same page shell keeps it looking the same (R5). |
| "We've sent a copy to {email}." is replaced by "We'll contact you at {email}." | Server email is a non-goal, so the original line would be untrue (FR-019). |
| A "Sending…" label and disabled controls while submitting | This state isn't in the design (spec, Principle II). |
| A submit-failure message above the actions | This state isn't in the design. It uses the error tint and error colour from the style sheet. |
| Two new CV rejection messages | The design defines no copy for a wrong file type or a file that is too big. |

## Project Structure

### Documentation (this feature)

```text
specs/001-job-application-form/
├── plan.md  research.md  data-model.md  quickstart.md
├── contracts/  routes.md  services.md  storage-schema.md
├── checklists/requirements.md
└── tasks.md        # created by /speckit-tasks
```

### Source Code

```text
frontend/
├── angular.json  package.json  eslint.config.js  .prettierrc
└── src/
    ├── index.html                  # Inter font link
    ├── styles.css                  # imports tokens.css and base.css
    ├── styles/tokens.css  base.css
    └── app/
        ├── app.config.ts  app.routes.ts  app.component.ts
        ├── core/storage/storage.service.ts (+ .spec.ts)
        ├── shared/ui/
        │   ├── button/button.component.ts
        │   ├── text-field/text-field.component.ts      # input + textarea, label/hint/error slot
        │   └── card/card.component.ts
        └── features/job-application/
            ├── job-application.routes.ts
            ├── has-application.guard.ts
            ├── data/northgate-senior-dotnet.role.ts
            ├── models/  job-role.model.ts  application.model.ts  application-draft.model.ts  form-view-state.model.ts
            ├── services/application.service.ts (+ .spec.ts)
            ├── validation/  validate-draft.ts  check-cv-file.ts  (+ specs)
            ├── state/  derive-form-state.ts  error-summary-text.ts  (+ specs)
            ├── formatting/  format-file-size.ts  first-name.ts  note-counter-text.ts  (+ specs)
            ├── pages/
            │   ├── apply-page/apply-page.component.ts                 # smart: draft signals, submit
            │   └── confirmation-page/confirmation-page.component.ts   # smart: reads latest()
            └── components/                                            # presentational
                ├── role-page-shell/   role-header/   role-facts/   role-description/
                ├── application-form/  error-summary/  cv-picker/
                └── confirmation-card/
```

**Structure Decision**: one Angular app in `frontend/`, with feature-first folders as the
constitution requires. `role-page-shell` wraps both pages through content projection, so the
confirmation route looks like the design's in-place swap.

## Complexity Tracking

No constitution violations. The R8 reading of "error summary linked to the fields" (announced
summary plus field-level aria, with no per-field anchor links) is an interpretation, not a
violation. Reviewers can overturn it.
