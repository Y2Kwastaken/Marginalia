# Topic: entities

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/entity-api/display-entities.md` | 256 | 1.20 | Text/item/block displays: transformation, scale, rotation, interpolation, billboarding |
| `paper/dev/api/entity-api/entity-teleport.md` | 91 | 1.21.10 | Teleporting: async teleport, `TeleportFlag`, passengers |
| `paper/dev/api/entity-api/mob-goals.md` | 139 | — | Custom mob AI — adding/removing goals via the Mob Goal API |
| `paper/dev/api/entity-api/entity-pathfinder.md` | 66 | — | Making a mob walk to a location |

## Notes

- **Teleporting is version-sensitive.** `entity-teleport.md` is marked 1.21.10; on older targets
  the flags and async variants differ. Check the user's version first.
- **Display entities are client-side visual only** — no collision, no physics. They are the
  right tool for holograms; armour-stand hologram hacks are obsolete.
- Interpolation on display entities needs both the duration and the start-delay set, and the
  transformation applied *after* — `display-entities.md` covers the ordering.
- Mob goals are Paper API, not NMS. Do not reach for internals here.
- Spawning entities in a Folia environment is region-threaded — see
  [scheduling-folia.md](scheduling-folia.md).

## Do not read

`paper/dev/api/entity-api/index.mdx` — 8-line stub.
