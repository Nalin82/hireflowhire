# Feature Specification: Job Application Form

**Feature Branch**: `001-job-application-form`

**Created**: 2026-10-01

**Status**: Draft

**Input**: User description: "Build the HireFlow Job Application screen. The visual design is in
./design/Job Application Page.dc - reference it, but this spec is about behaviour. Present the
application form for the advertised role; validate required fields and email format; show errors
inline; block submit until valid; on submit persist the application, then show a confirmation
state; handle a submit failure with a visible error and the ability to retry. Non-goals: auth, CV
file upload, multi-step wizard, server email, search."

**Design reference**: `design/Job Application Page.dc.html` (desktop) and
`design/Job Application Mobile.dc.html` (small screens). The design is the source of truth for
layout, copy and visual treatment. This spec defines behaviour only.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Apply for the advertised role (Priority: P1)

A candidate reading the "Senior .NET Engineer (Remote, UK)" role at Northgate Labs fills in their
full name, email and an optional cover note, sends the application, and sees a confirmation that
it has been received.

**Why this priority**: Submitting an application is the whole point of the screen. Without it,
the page is just a job advert.

**Independent Test**: Open the page, fill in valid details, select "Send application", and
confirm that the confirmation state appears and the application is stored.

**Acceptance Scenarios**:

1. **Given** the page has just loaded (default state), **When** the candidate views it, **Then**
   they see the role details (title, company, salary, location, contract, team, closing date,
   hiring manager), the role description, and the "Apply for this role" form with empty fields,
   hint text under each field and an enabled "Send application" button.
2. **Given** all required fields hold valid values, **When** the candidate selects "Send
   application", **Then** the form enters the submitting state: the button shows that sending is in
   progress, every field and the button are disabled, and selecting the button again doesn't send a
   second application.
3. **Given** the application has been saved, **When** submitting finishes, **Then** the form is
   replaced by the confirmation card ("Application sent"), which greets the candidate by first
   name and offers "Browse other roles" and "Start again".
4. **Given** a cover note is being typed, **When** its length changes, **Then** the counter shows
   "N / 500 characters" (or "Up to 500 characters." when empty), and input beyond 500 characters
   is not accepted.

---

### User Story 2 - Fix mistakes before sending (Priority: P1)

A candidate who leaves a required field empty, or mistypes their email address, is told clearly
what to fix, next to the field concerned, and can't send the application until it is valid.

**Why this priority**: Without validation, applications arrive incomplete or can't be answered.
Clear feedback is what makes a two-minute completion time realistic.

**Independent Test**: Select "Send application" with empty or invalid fields, check the error
summary and the inline messages, then correct the fields one at a time and watch each error clear.

**Acceptance Scenarios**:

1. **Given** the form is in the default state, **When** the candidate types in a field, **Then** no
   error messages appear yet. Fields aren't validated before the first attempt to send.
2. **Given** one or more required fields are empty or invalid, **When** the candidate selects "Send
   application", **Then** nothing is saved, the form enters the validating state, and an error
   summary appears above the fields. It reads "One thing needs fixing before you can send this."
   (one error) or "A few things need fixing before you can send this." (more than one), followed by
   "Check the highlighted fields below."
3. **Given** the validating state, **When** the candidate looks at an invalid field, **Then** that
   field is highlighted as in error and its hint is replaced by the specific message: "Enter your
   name." for full name, "Enter your email address." for an empty email, or "Add a domain, like
   name@company.com." for a malformed email.
4. **Given** the validating state, **When** the candidate corrects a field, **Then** that field's
   error clears as they type, and the summary updates its wording or disappears once no errors
   remain.
5. **Given** the validating state with every error resolved, **When** the candidate selects "Send
   application", **Then** submission proceeds exactly as in User Story 1.
6. **Given** a name or email made only of spaces, **When** the candidate tries to send, **Then** it
   is treated as empty.

---

### User Story 3 - Recover from a failed send (Priority: P2)

If the application can't be saved, the candidate sees a clear error, keeps everything they typed,
and can try again without starting over.

**Why this priority**: Failures are rare, but losing a written cover note is costly and erodes
trust. It is ranked below the core flow because the core flow must exist first.

**Independent Test**: Make the save fail (for example, with storage unavailable), send a valid
application, and confirm that the error state appears with the entered data intact. Then let the
save succeed and retry.

**Acceptance Scenarios**:

1. **Given** a valid form, **When** saving the application fails, **Then** the form enters the
   error state: a visible message above the actions explains the application wasn't sent and that
   their details are still here, all entered values are kept, and the fields become editable again.
2. **Given** the error state, **When** the candidate selects "Send application" again (retry),
   **Then** the form runs validation and submitting again, and on success shows the confirmation
   card.
3. **Given** the error state, **When** the candidate edits a field, **Then** the failure message
   stays visible until the next attempt to send.

---

### User Story 4 - Application survives a reload (Priority: P2)

A candidate who has sent an application and then reloads or returns to the page still sees that
their application was received. It isn't lost, and the blank form doesn't silently come back.

**Why this priority**: Persistence is an explicit acceptance criterion and protects candidates
from thinking their application vanished.

**Independent Test**: Submit an application, reload the page, and confirm that the saved
application is still there and the page reflects it.

**Acceptance Scenarios**:

1. **Given** an application has been sent for this role, **When** the page is reloaded, **Then**
   the confirmation card is shown again, with the same candidate first name taken from the saved
   application.
2. **Given** the confirmation card, **When** the candidate selects "Start again", **Then**
   [NEEDS CLARIFICATION: Q2 — does "Start again" only clear the form (the saved application is
   kept, and a second application can be sent), or does it withdraw/replace the saved one?]
3. **Given** no application has been sent, **When** the page is reloaded, **Then** the form is
   shown in the default state.

---

### Edge Cases

- **Double submission**: repeated activation of "Send application" while submitting MUST result
  in exactly one saved application.
- **Very fast save**: the submitting state MUST still be perceptible (shown for at least a brief
  moment) so the transition to confirmation doesn't feel like a glitch.
- **Pasted cover note over 500 characters**: it is cut off at 500 characters, and the counter shows
  "500 / 500 characters".
- **Leading or trailing spaces** in name and email are trimmed before validation and saving.
- **Email format**: the address must have text before an "@", a domain, and a dot within the
  domain (e.g. `name@company.com`). Addresses containing spaces are invalid.
- **Storage unavailable or full** (e.g. private browsing, quota exceeded): this is treated as a
  submit failure (User Story 3), never as a silent success.
- **Saved data that can't be read** on reload (corrupted or from an incompatible version): the page
  falls back to the default form instead of failing to load.
- **Closing date has passed**: the role shows "Applications close 29 August 2026". This feature
  doesn't enforce the closing date (see Assumptions).
- **Narrow screens**: every state works down to 360px wide.
- **Keyboard and screen-reader users**: the error summary is announced when it appears, focus
  moves to it (or to the first invalid field) on a failed attempt, and the confirmation card is
  announced when it replaces the form.

## Requirements *(mandatory)*

### Functional Requirements

**Content & default state**
- **FR-001**: The screen MUST show the advertised role's details exactly as in the design: company
  and team ("Northgate Labs · Platform"), title, summary line, salary, location, contract, team
  size, closing date, hiring manager (Dana Kowalski), and the full role description sections.
- **FR-002**: The application form MUST contain, in this order: Full name (required), Email
  (required), [NEEDS CLARIFICATION: Q1 — CV field], Cover note (optional, at most 500 characters),
  and a "Send application" action with the privacy note beneath it.
- **FR-003**: Each field MUST show its hint text from the design in the default state ("As it
  appears on your CV.", "We'll only use this about your application.", "Up to 500 characters.").
- **FR-004**: All copy MUST be the real copy from the design. Placeholder text MUST NOT appear.
  Copy that would be untrue for this feature MUST be adjusted (see FR-019).

**Validation**
- **FR-005**: Full name is required. A value with no visible characters MUST be rejected with
  "Enter your name."
- **FR-006**: Email is required ("Enter your email address.") and MUST be a valid format ("Add a
  domain, like name@company.com.").
- **FR-007**: Validation MUST first run when the candidate tries to send. Before that first
  attempt, no error messages are shown.
- **FR-008**: After a failed attempt, validation MUST re-run as the candidate edits any field, and
  each field's error, and the summary, MUST update straight away (the validating state).
- **FR-009**: Submission MUST be blocked while any field is invalid. The application is not saved
  and the candidate stays on the form. The "Send application" button stays available so that
  selecting it can reveal what needs fixing.
- **FR-010**: Errors MUST be shown inline under the field concerned, and as a summary above the
  fields that counts errors as "One thing" or "A few things".
- **FR-011**: Invalid fields MUST be marked as invalid to assistive technology, with the error
  message linked to the field. The summary MUST be announced when it appears.

**Submitting**
- **FR-012**: When a valid form is sent, the screen MUST enter a submitting state: the action shows
  progress (e.g. "Sending…"), fields and actions are disabled, and further sends are ignored.
- **FR-013**: The application MUST be saved before the confirmation is shown. The confirmation MUST
  NOT appear if saving failed.

**Success**
- **FR-014**: After a successful save, the form MUST be replaced by the confirmation card: the
  heading "Application sent", a thank-you that uses the candidate's first name (or "there" if no
  first name can be found), the review timeline, a "Browse other roles" link and a "Start again"
  action.
- **FR-015**: A saved application MUST still be there after a page reload or after the browser
  is closed and reopened on the same device and browser.
- **FR-016**: When a saved application exists for this role, loading the page MUST show the
  confirmation card instead of an empty form (User Story 4).
- **FR-017**: "Start again" behaviour: see User Story 4, scenario 2 (pending Q2).

**Error / retry**
- **FR-018**: If saving fails, the screen MUST enter an error state with a visible, plain-language
  message (e.g. "We couldn't send your application. Your details are still here. Please try
  again."). Everything the candidate entered is kept, the form becomes editable, and "Send
  application" works as a retry.

**Copy corrections for non-goals**
- **FR-019**: Because no email is sent, the confirmation MUST NOT claim that a copy was emailed.
  The design's sentence "We've sent a copy to {email}." MUST be removed or replaced by a truthful
  line (e.g. "We'll contact you at {email}."). This deviation from the design MUST be recorded in
  the plan.

### Key Entities

- **Job role (advertised position)**: the single role this screen applies to. Attributes: an
  identifier, company, team, title, summary, salary range, location, contract type, team size,
  closing date, hiring manager (name, role), and description sections. Read-only content.
- **Application**: one candidate's submission for a role. Attributes: an identifier, role
  identifier, full name (trimmed), email (trimmed), cover note (optional, 0–500 characters),
  [Q1: CV reference, if any], and the date and time it was submitted. It is created once per
  successful submit and kept on the candidate's device.
- **Form state**: exactly one of default, validating, submitting, success or error at any time,
  plus the current field values and any field errors.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A first-time candidate with their details to hand can go from page load to the
  confirmation card in under 2 minutes, without help (cover note left empty or short).
- **SC-002**: 100% of attempts to send with a missing name, missing email or malformed email are
  blocked, and every one shows at least one inline error naming the problem field.
- **SC-003**: 100% of successfully sent applications are still present after a page reload, and
  the page shows the confirmation for them.
- **SC-004**: Each of the five states (default, validating, submitting, success, error) can be
  reached and is visibly distinct, on screens 360px wide and up.
- **SC-005**: After a forced save failure, the candidate can retry and complete their application
  without retyping anything. Data retained across the failure: 100%.
- **SC-006**: Repeated activation of "Send application" while submitting creates exactly one
  application.

## Assumptions

- **Single role**: the screen serves one fixed advertised role (the Northgate Labs Senior .NET
  Engineer post from the design). Listing, choosing or searching roles is out of scope; "Browse
  other roles" and "All open roles" are plain links to a destination outside this feature.
- **Local persistence**: "persist" means saving on the candidate's own device and browser. There is
  no server or back end in this feature, so applications aren't visible to the hiring team yet.
  Where a save can fail, the failure comes from local storage being unavailable or full.
- **No authentication**: candidates are anonymous. A saved application is tied to the browser, not
  to an account.
- **Closing date not enforced**: the closing date (29 August 2026) and the review date
  ("by 5 September") are shown as design content only. Accepting or rejecting applications after
  the closing date is out of scope.
- **"Block submit until valid"** means the application can't be sent or saved while invalid. It
  doesn't mean disabling the button, which would hide why sending isn't possible. This follows the
  design.
- **Validating state** means the form is showing validation feedback after a failed attempt and
  re-checking as the candidate types. Every check runs instantly on the candidate's device, so
  there is no remote validation.
- **Name format**: any non-blank name is accepted. There are no rules on length or characters,
  beyond trimming.
- **Non-goals**: authentication, CV file upload, a multi-step wizard, sending emails from a server,
  and search are excluded.
