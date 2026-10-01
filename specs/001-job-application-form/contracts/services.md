# Contract: Services

Both services use `providedIn: 'root'`. These are signatures only; the behaviour is unit-tested.

## StorageService (`core/storage/storage.service.ts`)

```ts
type StorageWriteResult = { ok: true } | { ok: false; reason: 'quota' | 'unavailable' };

read<T>(key: string, guard: (value: unknown) => value is T): T | null;
// Returns null when the key is missing, the JSON can't be parsed (console.warn), or the guard fails.

write(key: string, value: unknown): StorageWriteResult;
// JSON.stringify + setItem. It never throws: QuotaExceededError maps to 'quota', anything else
// to 'unavailable'.
```

## ApplicationService (`features/job-application/services/application.service.ts`)

```ts
type SubmitResult = { ok: true; application: Application } | { ok: false };

readonly latest: Signal<Application | null>;   // most recent by submittedAt, read on construction

submit(draft: ValidApplicationDraft, role: JobRole): Promise<SubmitResult>;
// Builds the Application (trimmed fields, uuid, ISO time), appends it to the stored list, writes
// it, and resolves after at least 600ms. On failure nothing is appended and `latest` is unchanged.

all(): readonly Application[];                    // every valid saved entry, oldest first
```

`ValidApplicationDraft` is an `ApplicationDraft` with `cv: CvDetails` (not null). It is produced
only by `validateDraft` returning no errors.

## Pure functions (`features/job-application/`)

| Function | File | Purpose |
|---|---|---|
| `validateDraft(draft): ApplicationErrors` | `validation/validate-draft.ts` | the spec's FR-005 to FR-006a |
| `checkCvFile(file: { name; size; type }): CvDetails \| CvRejection` | `validation/check-cv-file.ts` | type and size rules |
| `deriveFormState(attempted, status): FormViewState` | `state/derive-form-state.ts` | research R4 |
| `errorSummaryText(count): string` | `state/error-summary-text.ts` | "One thing…" / "A few things…" |
| `formatFileSize(bytes): string` | `formatting/format-file-size.ts` | research R7 |
| `firstName(fullName): string` | `formatting/first-name.ts` | first word, or `'there'` |
| `noteCounterText(length): string` | `formatting/note-counter-text.ts` | "Up to 500 characters." / "N / 500 characters" |
