# Contract: Routes

| Path | Loads | Guard | Behaviour |
|---|---|---|---|
| `''` | n/a | n/a | redirect to `/apply` |
| `/apply` | `ApplyPageComponent` (lazy) | none | Always starts with an empty draft in the `default` state. On a successful save, navigates to `/confirmation` with `replaceUrl: true`. |
| `/confirmation` | `ConfirmationPageComponent` (lazy) | `hasApplicationGuard` | Shows the confirmation for `ApplicationService.latest()`. "Start again" navigates to `/apply`. |
| `**` | n/a | n/a | redirect to `/apply` |

`hasApplicationGuard: CanActivateFn` returns `true` when a saved application exists, and
otherwise `router.createUrlTree(['/apply'])`.

Both pages render the same page shell (site header, role header, description, facts panel and
footer). Only the slot for the form or confirmation card differs.

External links: "All open roles" and "Browse other roles" point to `/roles`. That route belongs to
a future Open Roles feature, so until it exists the `**` route sends it back to `/apply`.
