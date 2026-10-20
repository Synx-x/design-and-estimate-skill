# Known-stack notes

Optional shortcuts for repos you have already mapped. Read an entry only after
the Step 2 discovery pass has identified the repo, and apply only the matching
entry. Discovery always wins on conflict, because these notes go stale.

Each entry names how to detect the repo, then lists what discovery would
otherwise have to rediscover. Mark anything unverified as such.

---

## Example entry (replace with your own)

Detect by: a solution file next to projects named `Acme.*`.

| Project | Role |
|---------|------|
| Acme.Web | MVC frontend |
| Acme.Api | HTTP API, owns the controllers |
| Acme.Data | ORM entities and migrations |

Notes worth recording: sibling folders that sit outside the solution, the
permission or policy constants in use, how menu entries get registered, and the
UI component library.
