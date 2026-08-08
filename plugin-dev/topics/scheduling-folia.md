# Topic: scheduling and Folia

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/scheduler.md` | 220 | — | Any delayed, repeating, or async task |
| `paper/dev/api/folia-support.md` | 84 | — | Making a plugin work on Folia — read **before** writing schedulers if Folia is a target |
| `folia/admin/reference/region-logic.md` | 291 | — | Understanding *why* Folia's threading constrains you; how regions split and merge |
| `folia/admin/reference/overview.md` | 304 | — | Broader Folia architecture; usually more than a plugin dev needs |

## Notes

- **20 ticks = 1 second**, and ticks slip when the server lags. A "100 tick" delay is *at least*
  5 seconds, not exactly.
- **Never `Thread.sleep` on the main thread.** That is a server-wide freeze.
- **Async tasks may not touch most of the Bukkit API.** Do the slow thing async, then schedule
  back onto the main thread to apply results.
- **Folia changes everything about scheduling.** There is no single main thread — there are
  region threads, plus a global region thread. `Bukkit.getScheduler()` is not valid there.
  `folia-support.md` covers the `RegionScheduler` / `EntityScheduler` / `AsyncScheduler` /
  `GlobalRegionScheduler` split. If Folia support is wanted, read that **first** — retrofitting
  it later means rewriting every task.
- A plugin declares Folia support explicitly; without the declaration Folia refuses to load it.

## Do not read

`folia/index.mdx`, `folia/admin/index.mdx`, `folia/admin/reference/index.mdx` — 10–16 line stubs.
`folia/admin/reference/faq.md` (74) is admin-facing, not development.
