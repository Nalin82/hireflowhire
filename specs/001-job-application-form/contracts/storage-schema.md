# Contract: localStorage schema

- **Key**: `applications`
- **Value**: a JSON array of `Application` (see [data-model.md](../data-model.md)), oldest first.
  Each send appends one entry.

```json
[
  {
    "id": "6f1c2a8e-3b7d-4c0e-9a51-2d8f4e7b9c13",
    "jobId": "northgate-senior-dotnet-remote-uk",
    "jobTitle": "Senior .NET Engineer (Remote, UK)",
    "fullName": "Ravi Shah",
    "email": "ravi.shah@fastmail.com",
    "coverNote": "I've spent the last four years moving a booking monolith onto .NET 8 services.",
    "cv": { "fileName": "Ravi-Shah-CV.pdf", "sizeBytes": 421888, "mimeType": "application/pdf" },
    "submittedAt": "2026-10-01T09:42:17.000Z"
  }
]
```

| Situation on read | Behaviour |
|---|---|
| key missing | treated as `[]` |
| not valid JSON, or not an array | treated as `[]`, with a `console.warn`. The next successful write replaces it. |
| an array containing invalid entries | invalid entries are skipped and valid ones kept |

The stored value has no schema version. Adding one is deferred until a second version exists.
