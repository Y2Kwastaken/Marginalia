# Topic: configuration, storage, and messaging

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/plugin-configs.mdx` | 173 | — | `config.yml`, defaults, `saveDefaultConfig`, reload behaviour |
| `paper/dev/api/pdc.md` | 271 | 1.21.8 | Attaching persistent data to items, entities, blocks, chunks |
| `paper/dev/misc/databases.md` | 219 | — | SQL/database storage, connection pooling, doing it off the main thread |
| `paper/dev/api/plugin-messaging.md` | 228 | — | Talking to a proxy (Velocity/BungeeCord) or the client over plugin channels |

## Choosing where to put data

- **A few settings the admin edits** → `config.yml` (`plugin-configs.mdx`).
- **Data attached to a specific item/entity/block** → PDC. This is the right answer far more
  often than people think, and it survives restarts and item movement.
- **Lots of rows, queried** → a database (`databases.md`).

## Notes

- **Database and file I/O must not run on the main thread.** Every tick spent waiting on a query
  is a tick the server does not do. Use the async scheduler
  ([scheduling-folia.md](scheduling-folia.md)) and hop back to the main thread to touch the world.
- **PDC keys are `NamespacedKey`s** owned by your plugin — namespace them, do not collide.
- **`getConfig()` caches.** After editing the file on disk you need `reloadConfig()`.
- Plugin messaging requires registering the channel in `onEnable` on both ends; a Velocity-side
  counterpart is Velocity's own docs, out of scope for this skill.
