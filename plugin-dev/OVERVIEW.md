# Prong 1: Paper plugin development

Reached from [../SKILL.md](../SKILL.md).

## Step 0 — set up ground truth

The PaperMC docs site is this prong's source of truth, cloned into the project at
`.claude/extern/papermc-docs/`. **Do not answer Paper API questions from memory.**

Once per session, before the first lookup:

1. **[bootstrap.md](bootstrap.md)** — clone the docs if missing, refresh if the last fetch was
   over 30 days ago, and what to do when upstream moves a file. This writes to disk, so it must
   happen **before or outside plan mode**: if the clone is missing while you are planning, leave
   plan mode and bootstrap first rather than planning a Paper task from memory.
2. **[conventions.md](conventions.md)** — how to read these docs without producing broken
   output: version placeholders are literal in the files, `jd:` links are not Java, cross-links
   use slugs that never match file paths, and the `:::caution` blocks carry the real warnings.

All doc paths in this prong are relative to
`.claude/extern/papermc-docs/src/content/docs/`.

## Core rules

Always true, and cheaper to carry than to look up.

**Plugin structure**
- The main class extends `JavaPlugin`. Do not name it `Main` — name it after the plugin.
- **No logic in the main class constructor** — no guarantee any API is available there. Never
  call the constructor yourself.
- Lifecycle: `onLoad` (most of the Bukkit API is unavailable) → `onEnable` (register listeners;
  safe for DB connections and threads) → `onDisable` (cleanup, save, close).
- Use `plugin.yml`. Only use `paper-plugin.yml` when the user asks for it or needs a
  bootstrapper/loader.

**Wrong by default**
- Text is Adventure `Component`. Never legacy `§` or `&` colour codes.
- Commands are Brigadier. `CommandExecutor`/`onCommand` is the legacy path — only touch it when
  modifying existing code that already uses it.
- Delays use the scheduler, never `Thread.sleep` on the main thread. 20 ticks = 1 second, and
  ticks slip under server lag.
- Never hardcode `api-version` or the `paper-api` version. Resolve them — see
  [conventions.md](conventions.md) §1.
- Need an exact signature the docs do not give? `javap -p` the `paper-api` jar already in the
  gradle cache — never `jar -xf` it. See [conventions.md](conventions.md) §5.

**Performance**
- Hot events (`PlayerMoveEvent`, `BlockPhysicsEvent`) fire constantly. Cheapest check first,
  early-return, then do work.

**On the "Experimental" label**
Paper marks a lot of long-stable API `@Experimental` and rarely removes the label, so it is a
weak signal *for a plugin author deciding whether to depend on something*. Read it as *"the
signature may change,"* not *"do not use."* Mention it to the user once when it applies, then use
the API normally — do not refuse it or route around it into a worse pattern. Only three docs
carry the badge, but other explicit APIs may: `registries.md`, `lifecycle/datapacks.mdx`,
`getting-started/paper-plugins.md`.

## Router — open one topic file, not all of them

| Task | Read |
|---|---|
| New project, Gradle/Maven setup, `plugin.yml`, plugin basics | [topics/getting-started.md](topics/getting-started.md) |
| Commands, arguments, tab completion, permissions on commands | [topics/commands.md](topics/commands.md) |
| Listening to events, custom events, chat | [topics/events.md](topics/events.md) |
| Messages, colour, MiniMessage, titles, boss bars, sound, books | [topics/text-adventure.md](topics/text-adventure.md) |
| GUIs, inventories, items, item data components, recipes | [topics/inventories-items.md](topics/inventories-items.md) |
| Entities, display entities, mob AI, teleportation | [topics/entities.md](topics/entities.md) |
| Config files, PDC, databases, plugin messaging | [topics/data-persistence.md](topics/data-persistence.md) |
| Scheduling, async work, Folia region threading | [topics/scheduling-folia.md](topics/scheduling-folia.md) |
| Lifecycle API, registries, datapacks, NMS/internals | [topics/lifecycle-registries.md](topics/lifecycle-registries.md) |
| Particles, dialogs | [topics/visual-ux.md](topics/visual-ux.md) |
| Debugging, stacktraces, profiling, server config | [topics/debugging.md](topics/debugging.md) |
| Publishing to Hangar, downloads API | [topics/publishing.md](topics/publishing.md) |

## Digests

Two upstream docs are pre-condensed because they are mostly scaffolding. Prefer these:

- [digests/minimessage-format.md](digests/minimessage-format.md) — every MiniMessage tag as one
  table. Replaces a 577-line read.
- [digests/adventure-essentials.md](digests/adventure-essentials.md) — Components, Audiences,
  MiniMessage setup, placeholders, serializers. Replaces ~530 lines across four files.

Each digest names its upstream sources and their blob SHAs. If a digest does not cover what you
need, open the upstream file it names.
