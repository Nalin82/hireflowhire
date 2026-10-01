# Data Model: Job Application Form

Every model is a TypeScript interface or type alias in
`frontend/src/app/features/job-application/models/`.

## JobRole (read-only content)

The single advertised role. It is a typed constant, `NORTHGATE_SENIOR_DOTNET`, holding the copy from
`design/Job Application Page.dc.html` word for word.

| Field | Type | Example |
|---|---|---|
| id | `string` | `'northgate-senior-dotnet-remote-uk'` |
| company | `string` | `'Northgate Labs'` |
| team | `string` | `'Platform'` |
| title | `string` | `'Senior .NET Engineer (Remote, UK)'` |
| summary | `string` | `'Own the .NET services behind our scheduling and messaging products.'` |
| salary | `string` | `'£78,000 – £92,000'` |
| location | `string` | `'Remote, UK'` |
| contract | `string` | `'Permanent, full time'` |
| teamSize | `string` | `'Platform, 9 people'` |
| closingDate | `string` (ISO date) | `'2026-08-29'`, shown as "29 August 2026" |
| hiringManager | `{ name: string; role: string; initials: string }` | Dana Kowalski, Hiring manager, DK |
| description | `readonly RoleSection[]` | `{ heading: string \| null; paragraphs: readonly string[] }` |
| reviewNote | `string` | Confirmation copy: "…review applications every Thursday, and you'll hear from us either way by 5 September." |

## CvDetails

| Field | Type | Rule |
|---|---|---|
| fileName | `string` | non-empty |
| sizeBytes | `number` | 1 to 10,485,760 |
| mimeType | `string` | the file's type, which may be `''`. Accepted when the extension is `.pdf`, `.doc` or `.docx` **or** the MIME type is `application/pdf`, `application/msword` or `application/vnd.openxmlformats-officedocument.wordprocessingml.document` |

## ApplicationDraft (form values, never saved as-is)

| Field | Type | Validation (applied after trimming) | Message |
|---|---|---|---|
| fullName | `string` | required | "Enter your name." |
| email | `string` | required, then `/^[^\s@]+@[^\s@]+\.[^\s@]+$/` | "Enter your email address." / "Add a domain, like name@company.com." |
| cv | `CvDetails \| null` | required | "Attach your CV to apply." |
| coverNote | `string` | optional, cut off at 500 characters on input | n/a |

`ApplicationErrors = Partial<Record<'fullName' | 'email' | 'cv', string>>`

**CV selection errors** are separate from validation on send. They show as soon as a file is
chosen: "Choose a PDF, DOC or DOCX file." and "That file is over 10MB. Try a smaller version." They
are held in a `cvRejection` signal, which is cleared on the next valid choice or on Remove.

## Application (saved)

Reconciled from the plan input `{ id, jobTitle, fullName, email, phone, coverNote, submittedAt }`
against the page:

| Field | Type | Note |
|---|---|---|
| id | `string` | `crypto.randomUUID()` |
| jobId | `string` | **added** so "this role" can be identified without matching on title text |
| jobTitle | `string` | copied from the `JobRole.title` |
| fullName | `string` | trimmed |
| email | `string` | trimmed |
| coverNote | `string` | `''` when empty |
| cv | `CvDetails` | **added**: the design's required CV field (Q1-B) |
| submittedAt | `string` | ISO-8601 UTC |
| ~~phone~~ | n/a | **dropped**: not on the page |

`isApplication(value: unknown): value is Application` is the type guard used on read.

## Form state

```text
submitStatus: 'idle' | 'submitting' | 'failed' | 'saved'
attempted:    boolean            (true after the first attempt to send)
draft:        ApplicationDraft
errors      = computed(attempted ? validateDraft(draft) : {})
viewState   = computed(deriveFormState(attempted, submitStatus))  → FormViewState
```

Transitions:

```text
default ──send (invalid)──▶ validating ──edit──▶ validating (errors update live)
default/validating ──send (valid)──▶ submitting ──save ok──▶ success ──▶ /confirmation
                                         └──save fails──▶ error ──send──▶ submitting …
/confirmation ──Start again──▶ /apply (default, new empty draft)
```

## Storage

See [contracts/storage-schema.md](contracts/storage-schema.md).
