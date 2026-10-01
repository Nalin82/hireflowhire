# Quickstart: Job Application Form

## Prerequisites

Node 24 LTS and npm 10. Every command runs **inside `frontend/`**.

```bash
npm install
npm start          # ng serve → http://localhost:4200/apply
npm test           # ng test (Vitest), unit tests only
npm run lint       # ng lint (angular-eslint)
npm run format:check
```

## Manual acceptance checks (constitution IV: record each one in tasks.md)

Run each check at 360px and at 1040px or wider, and compare against
`design/Job Application Page.dc.html`.

| # | Steps | Expected |
|---|---|---|
| M1 Default | Open `/apply` | Role content, empty form, hints showing, no errors |
| M2 Validating | Select "Send application" with everything empty | Summary "A few things need fixing…", 3 red fields, focus on Full name |
| M3 Live re-check | Type a name, then `ravi@fastmail` | Name error clears, email shows "Add a domain…", and the summary still says "A few things…" (email and CV). Fix the email: the summary says "One thing…". Add the CV: the summary disappears |
| M4 CV rules | Drop a `.png`, then a file over 10MB, then a valid PDF | The two rejection messages appear in turn, then the file name and size with "Remove" |
| M5 Submitting | Send a valid form | "Sending…", controls disabled, about 600ms; double-clicking saves only one entry |
| M6 Success | After the send | `/confirmation`, "Thanks Ravi", no claim that an email was sent |
| M7 Reload | Reload `/confirmation` | Same card, with data from storage |
| M8 Guard | Clear storage, then open `/confirmation` | Redirected to `/apply` |
| M9 Failure + retry | In DevTools run `Storage.prototype.setItem = () => { throw new DOMException('full', 'QuotaExceededError'); }`, then send | Error message showing, input kept. Reload the page to restore storage, fill in again, send: success |
| M10 Start again | Select "Start again" | Empty `/apply`. `JSON.parse(localStorage.applications).length` is unchanged |
| M11 Keyboard/AT | Complete the whole flow using only the keyboard, with a screen reader on | Visible focus ring, summary announced, errors read out with their fields |
| M12 Two minutes | Time a first-time run from load to confirmation | Under 2 minutes (SC-001) |
