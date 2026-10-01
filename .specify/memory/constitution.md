<!--
Sync Impact Report
- Version change: 1.0.0 → 2.0.0
  MAJOR because Principle IV is redefined: UI/component and end-to-end automation tests are no
  longer required for this workshop. Acceptance criteria are verified by unit tests of core logic
  and services, plus a manual check against the design. The new structure, naming and component
  rules would on their own be a MINOR bump.
- Modified principles:
  III. Standalone Angular, Strictly Typed → III. Angular Architecture & Conventions (expanded:
       frontend/ location, feature-first folders, lazy routes, naming, hf- prefix, OnPush, signal
       APIs, built-in control flow, signal state, root services, observable hygiene)
  IV. Acceptance Criteria Are the Definition of Done (redefined test scope, see above)
  I. Design Fidelity via Tokens (clarified: component-scoped styles)
- Added sections: Repository Layout (under Development Workflow), Quality Gates
- Removed sections: none
- Templates requiring updates:
  ✅ .specify/templates/plan-template.md (Constitution Check reads this file at runtime; no edit needed)
  ✅ .specify/templates/spec-template.md (no edit needed)
  ⚠ .specify/templates/tasks-template.md (its sample tasks include contract and integration (user-journey) tests;
     /speckit-tasks MUST leave out UI/e2e tasks under Principle IV. Template left unchanged.)
- Follow-up TODOs (carried from 1.0.0):
  - The user referred to "HireFlow Visual Language.dc"; the file on disk is
    design/HireFlow Style Sheet.dc.html (page title "Visual language").
  - Job Application Page.dc.html renders default, error and success states only. Validating and
    submitting states MUST be defined in the spec (Principle II).
  - The style sheet has no success/warning colour role. Until one exists, success states use
    primary blue and neutrals only.
  - TODO(ANGULAR_VERSION): the Angular major version is not chosen yet. It is fixed when frontend/ is
    scaffolded, and the "official style guide" in Principle III means that version's guide.
-->

# HireFlow Constitution

## Core Principles

### I. Design Fidelity via Tokens

- `design/HireFlow Style Sheet.dc.html` (the "Visual language" sheet) is the single source of truth
  for colour, type scale, spacing, radius and the core components (button, text input, card).
- Every colour, font size, line height, spacing step, radius, border width and focus-ring value
  MUST be expressed as a CSS custom property (e.g. `--hf-color-primary`, `--hf-space-2`) defined
  once in a global tokens stylesheet. Styles MUST be component-scoped and MUST reference tokens.
  Hard-coded hex, px or rem literals in component styles are forbidden (exception: `0`).
- Token values MUST match the style sheet exactly. New tokens MUST NOT be invented without first
  adding them to the design source.
- `design/Job Application Page.dc.html` (with `Job Application Mobile.dc.html` for small
  viewports) MUST be rebuilt faithfully: same layout, hierarchy, copy, field order, hints, error
  messages and breakpoints (≤420px compact, ≥900px two-column with sticky facts panel).
  Deviations MUST be recorded in the feature's plan with a reason.

Rationale: tokens keep the build traceable to the design and make brand changes a one-file edit.

### II. Every Designed State Ships

- A feature is not done when only the happy path works. Every UI state MUST be implemented and
  reachable: at minimum **default, validating, submitting, success and error** for any form.
- States shown in the design MUST match it (e.g. the error summary above the fields, red field
  borders and inline messages, live re-validation after the first failed submit, the
  confirmation card after a successful send).
- States the design does not show (currently validating and submitting) MUST be described in the
  spec and built from existing tokens and components. Controls MUST be disabled and the double
  submit prevented while submitting, and storage or network failures MUST surface a recoverable
  error state.

Rationale: users reach unhappy paths often; an unbuilt state is a defect, not a gap.

### III. Angular Architecture & Conventions

Follow the official Angular style guide for the version in `frontend/package.json`, and apply it
consistently. The rules below are non-negotiable.

**Structure**
- Feature-first folders: a feature's components, service and models live together under
  `src/app/features/<feature>/`. Reusable presentational UI (button, text input, card) lives in
  `src/app/shared/`. App-wide singletons (storage, config, error handling) live in `src/app/core/`.
- Feature routes MUST be lazy-loaded (`loadComponent` / `loadChildren`).
- Standalone components, directives and pipes only. No NgModules.

**Naming**
- Files are kebab-case, with one concept per file. The filename matches its main export
  (`application-form.component.ts` → `ApplicationFormComponent`, or the suffix-free form if the
  chosen Angular version's guide uses it, applied the same way everywhere).
- Classes and model interfaces are PascalCase. Models are interfaces or type aliases, not classes,
  unless they carry behaviour.
- Component selectors use the `hf-` prefix (e.g. `hf-application-form`). Directive selectors use
  `hf` in camelCase (e.g. `hfAutofocus`).

**Components**
- `changeDetection: ChangeDetectionStrategy.OnPush` on every component.
- Use the signal APIs `input()`, `output()` and `model()`, never the `@Input`/`@Output`
  decorators.
- Keep components small. Separate smart (data, services, orchestration) components from
  presentational ones (inputs in, outputs out) where that makes things clearer.
- No business logic in templates: validation, formatting and derived values belong in `computed()`
  signals, pure functions or services.

**State & data**
- Hold local state in `signal()` and derive with `computed()`. Never store a value that can be
  derived from another one.
- Model UI state as an explicit typed union (e.g.
  `'default' | 'validating' | 'submitting' | 'success' | 'error'`), not as loose booleans.
- Services use `providedIn: 'root'`. All data access goes through a service (e.g.
  `ApplicationService`, `StorageService`); components never touch `localStorage`, `fetch` or
  `HttpClient` directly.
- TypeScript `strict` and Angular `strictTemplates` MUST be on. Typed models throughout; `any` is
  forbidden (use `unknown` plus narrowing at boundaries). Forms MUST be typed.

**Templates & reactivity**
- Use the built-in control flow (`@if`, `@for` with `track`, `@switch`), never `*ngIf`/`*ngFor`.
- Prefer signals. Where an observable is unavoidable, consume it with the `async` pipe or
  `toSignal()`. Manual subscriptions MUST be cleaned up with `takeUntilDestroyed()`.

Rationale: one consistent, modern Angular idiom keeps the codebase small and predictable, and
OnPush plus signals keep change detection cheap and explicit.

### IV. Acceptance Criteria Are the Definition of Done

- Every user story in a spec MUST have Given/When/Then acceptance criteria that cover each state in
  Principle II. A story is done only when every criterion is met.
- Core logic and services (validators, state derivation, `ApplicationService`, `StorageService`,
  formatting helpers) MUST have unit tests that pass.
- **Workshop time-box:** UI/component automation tests and end-to-end tests are deliberately
  skipped for this project. Acceptance criteria that can only be seen in the UI (states, layout,
  visual parity, accessibility) MUST be checked by hand against the design files, with each check
  recorded in the task list. This is a session constraint, not a default: a production client build
  MUST bring back component and e2e coverage, through a constitution amendment.

Rationale: shared, testable criteria define "done". The time-box saves time without dropping the
check that the build matches the design.

### V. Real Content Only

- Lorem ipsum and other placeholder copy are forbidden in code, fixtures, tests and demos.
- Use the real copy from the design (e.g. the Northgate Labs "Senior .NET Engineer (Remote, UK)"
  role, field hints and error messages) or realistic, domain-accurate content written in the
  HireFlow voice: warm and professional, not corporate-stiff.

Rationale: real content shows up layout, length and tone problems that filler text hides.

## Brand & Accessibility Constraints

- Primary blue `#1D4ED8` (hover/pressed `#1E40AF`), warm grey neutrals, Inter, 8px spacing grid.
  All of these are exposed only as tokens (Principle I).
- WCAG 2.1 AA contrast for all text and interactive states. Visible focus ring: 2px offset ring in
  primary blue.
- Semantic HTML (landmarks, headings in order, `<button>` for actions, `<form>` for forms). Every
  control has an associated `<label>`.
- Mobile-first; layouts MUST work down to 360px wide. Touch targets are 48px tall; buttons are
  40px tall on desktop.
- No gradients and no heavy drop shadows.
- Form errors MUST be announced to assistive technology (an error summary linked to the fields,
  `aria-invalid`, and hints wired up with `aria-describedby`).

## Development Workflow

**Repository layout**
- `frontend/`: the Angular application. All `ng`, `npm` and lint/test commands run from inside
  `frontend/`.
- `design/`: the exported design. It is read-only reference material and MUST NOT be imported by,
  or bundled into, the application.
- `.specify/`, `specs/`, `.claude/`: Spec Kit artifacts. They are kept separate from application
  code.

**Process**
- Features follow the Spec Kit cycle: specify → clarify (optional) → plan → tasks → implement.
- Each plan's Constitution Check MUST confirm: token coverage (no hard-coded values), the list of
  states with their design source or spec definition, conformance with the Principle III
  conventions, the mapping from acceptance criteria to unit tests or manual checks, and no
  placeholder content.
- Any request that conflicts with the brand rules MUST be flagged before it is built.

**Quality gates**
- ESLint with `angular-eslint` and Prettier, configured in `frontend/` and applied to every file.
  The lint config MUST enforce the `hf` selector prefix, ban `any`
  (`@typescript-eslint/no-explicit-any`), and require OnPush and the signal input/output APIs
  where rules exist for them.
- A change is mergeable only when lint, Prettier check, type-check/build and unit tests all pass.
- Code review MUST reject changes that add hard-coded style values, skip a required state, use
  `any`, break the Principle III conventions, or contain lorem ipsum.

## Governance

- This constitution overrides other project conventions. Where `CLAUDE.md` or design notes
  conflict with it, this document wins until it is amended.
- Amendments are made with `/speckit-constitution`, recorded in the Sync Impact Report and
  committed with a message that names the version.
- Versioning follows semantic versioning. MAJOR means a principle is removed or redefined in a way
  that breaks existing work. MINOR means a principle or section is added or materially expanded.
  PATCH means clarifications and wording only.
- Compliance is checked at plan time (Constitution Check) and at review. Any exception MUST be
  justified in the plan's Complexity Tracking table.

**Version**: 2.0.0 | **Ratified**: 2026-10-01 | **Last Amended**: 2026-10-01
