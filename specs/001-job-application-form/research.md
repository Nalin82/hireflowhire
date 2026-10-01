# Research: Job Application Form

Phase 0 decisions. Each one gives the decision, the reason and the alternatives considered.
Standing rules (tokens, OnPush, signals, naming, lint) live in the constitution and are not repeated
here.

## R1. Angular version and rendering mode

- **Decision**: Angular **22.2** (the current `latest` on npm, checked 2026-10-01), created with
  `ng new frontend --ssr=false`, browser-only, zoneless (the default since v21).
- **Rationale**: **Pinned as load-bearing: no SSR.** `localStorage` is only touched in the browser,
  so `StorageService` doesn't need `isPlatformBrowser` guards or a server fallback. Zoneless suits
  the signals plus OnPush approach.
- **Alternatives**: SSR/prerender was rejected because it isn't needed for one form page and would
  force platform guards around storage.
- **Consequence**: if SSR is ever enabled, `StorageService` MUST gain a platform guard first. That
  needs an amendment to this plan.

## R2. File naming under the v22 style guide

- **Decision**: keep the explicit type suffixes, `*.component.ts` / `*Component` and
  `*.service.ts` / `*Service`, set in the CLI schematics in `angular.json` so that `ng generate`
  emits them. Other types: `*.model.ts` (interfaces), `*.routes.ts`, `*.ts` for pure functions.
- **Rationale**: the constitution names `ApplicationService` and `StorageService`. The suffix-free
  v20+ default would give inconsistent names (`Application` reads like a model). Using suffixes
  everywhere keeps the "filename matches export" rule unambiguous.
- **Alternatives**: suffix-free naming (the v20+ default) was rejected because it clashes with the
  service names and model names.

## R3. Form handling

- **Decision**: **signals, without Angular Forms.** The smart page holds a `signal<ApplicationDraft>`
  plus `attempted` and `submitStatus` signals. Errors and the five-way view state are `computed()`
  from pure functions (`validateDraft`, `deriveFormState`) that are unit-tested on their own. Inputs
  bind with `[value]` and `(input)`.
- **Rationale**: there are four fields, so a forms framework adds little. Pure functions hold the
  business logic and are where the unit tests go (constitution IV). Everything stays signal-native
  with no observables, and the state is never duplicated.
- **Alternatives**: typed reactive forms were rejected because they are observable-based, need
  `toSignal` bridging, and would duplicate validation state. Experimental signal forms were
  rejected because their API may still change.

## R4. View state model

- **Decision**: `type FormViewState = 'default' | 'validating' | 'submitting' | 'success' | 'error'`,
  derived as follows:

  | Condition | State |
  |---|---|
  | `submitStatus === 'submitting'` | submitting |
  | `submitStatus === 'failed'` | error (validation errors also shown live) |
  | `submitStatus === 'saved'` | success (just before navigating to `/confirmation`) |
  | `attempted` and idle | validating (the summary is hidden once no errors remain) |
  | otherwise | default |

- **Rationale**: this matches the spec's definitions (validating means live re-checking after the
  first failed attempt). With one derived source, the view can't show two states at once.

## R5. Routes and reload behaviour

- **Decision**: `''` → redirect to `apply`. `apply` lazy-loads `ApplyPageComponent`.
  `confirmation` lazy-loads `ConfirmationPageComponent`, guarded by a functional `CanActivateFn`
  that redirects to `/apply` when no application is saved. `**` → redirect to `apply`. After a save,
  the app navigates to `/confirmation` with `replaceUrl: true`, so Back doesn't land on a half-sent
  form.
- **Rationale**: these are the routes from the plan input. The confirmation always reads the latest
  saved application, so no id appears in the URL (personal data stays out of URLs) and reload works
  without router state.
- **Design deviation**: in the design, the confirmation card replaces the form *in place*. Here it
  is a separate route that renders the **same page shell** (role header, description, facts panel)
  with the confirmation card in the form slot. It looks the same, but the URL changes. See
  plan.md › Design Deviations.

## R6. Persistence and failure

- **Decision**: `StorageService` wraps `localStorage` with typed `read<T>(key, guard)` and
  `write(key, value)` methods. `write` returns a result union and never lets an exception through.
  `ApplicationService` owns the `applications` key: it reads, validates each entry with a type
  guard, appends, and writes. Ids come from `crypto.randomUUID()` and timestamps are ISO-8601 UTC.
- **Failure sources**: `QuotaExceededError` and `SecurityError` (storage blocked). Both map to the
  spec's error state. Unreadable or invalid JSON on read is treated as an empty list, and a
  `console.warn` names the key. Malformed entries inside a valid array are skipped.
- **Minimum submitting time**: `ApplicationService.submit` resolves after at least **600ms**
  (spec edge case: the submitting state must be perceptible). Unit tests use fake timers.
- **Alternatives**: IndexedDB was rejected as heavier than needed for a few small records. A mock
  HTTP back end was rejected because "no server" is a non-goal.

## R7. CV handling (spec clarification Q1, option B)

- **Decision**: a hidden `<input type="file" accept=".pdf,.doc,.docx">` plus a drop zone. When a
  file is chosen, it is checked by extension and MIME type and against the 10MB limit, and only
  `{ fileName, sizeBytes, mimeType }` is kept. The `File` object is never stored or read.
- **Size label**: below 1 MiB it shows `max(1, round(bytes/1024))` KB, otherwise `bytes/1048576` to
  1 decimal place plus " MB". This matches the design's logic and lives in a pure formatter that is
  unit-tested.
- **Accessibility**: the drop zone is a real `<button>` ("Choose a file"). Drag and drop is extra,
  so keyboard users never depend on it.

## R8. Error summary and focus

- **Decision**: the summary keeps the design's copy and has `role="alert"` and `tabindex="-1"`. On a
  failed attempt, focus moves to the **first invalid field**, and the summary is announced through
  `role="alert"`. Each field gets `aria-invalid` and `aria-describedby` pointing at its hint/error
  slot. The submit-failure message also uses `role="alert"`.
- **Rationale**: the constitution requires "an error summary linked to the fields". This plan meets
  that through the announced summary plus field-level wiring, **without** adding per-field links to
  the summary, which would change the design's look. If reviewers read "linked" as per-field anchor
  links, the summary needs a visible list of links, and that becomes a design deviation.

## R9. Token sources and gaps

- **Decision**: put the tokens in `frontend/src/styles/tokens.css`. The primary source is
  `design/HireFlow Style Sheet.dc.html`: 11 colour roles, 7 type styles and 7 spacing steps, with a
  6px control radius, an 8px card radius, a 1px border, the focus halo and the 32/40/48px control
  heights.
- **Values found only in the page design** (`Job Application Page.dc.html`). They are design-sourced,
  so they are allowed as tokens, but each one is marked `/* page-sourced */`:
  - `#A8A29B` as the hover border. It is also used in the style sheet's live hover states but
    missing from its colour list.
  - The focus halo `rgba(29,78,216,0.32)` (style sheet: "3px blue halo").
  - Font sizes 10, 15, 28 and 40px (40px is the page title at 900px and wider, 28px at 420px and
    narrower). Line heights 1.15 and 1.7. Radii 16px and 20px (avatar or pill).
  - Layout widths: 1040px page maximum, 280px facts column, 56px column gap, breakpoints at 420px
    and 900px.
- **No success colour**: the success mark uses the primary tint and primary blue, as in the design.

## R10. Tooling

- **Decision**: `ng add angular-eslint`, plus Prettier (`prettier`, `eslint-config-prettier`). Unit
  tests use the default v22 runner (**Vitest** via `ng test`). There are no component or e2e tests
  (constitution IV time-box).
- **Lint rules to enable**: `@angular-eslint/component-selector` (prefix `hf`, kebab-case),
  `@angular-eslint/directive-selector` (prefix `hf`, camelCase),
  `@angular-eslint/prefer-on-push-component-change-detection`, `@angular-eslint/prefer-signals`,
  `@angular-eslint/template/prefer-control-flow`, `@typescript-eslint/no-explicit-any`.
- **Hard-coded style values**: there is no off-the-shelf rule for these, so they are enforced in
  review. Optional: `stylelint` with `color-no-hex` scoped to component styles. Deferred, not
  required.
