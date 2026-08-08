---
name: marginalia
description: PaperMC work — writing, building, and debugging Paper/Spigot/Bukkit server plugins (plugin.yml, Brigadier commands, event listeners, Adventure/MiniMessage, inventories, PDC, scheduler, Folia, particles, paper-api Gradle setup), and contributing patches to PaperMC itself. Use for any Paper, Spigot, Bukkit, Folia, Velocity, or Adventure question.
---

# marginalia

PaperMC work splits into two jobs that look similar and are not. **Decide which one this is
before doing anything else** — the APIs, the source of truth, and even which repository you read
are all different.

| You are… | Signals | Go to |
|---|---|---|
| **Writing a plugin** — code that runs *on* a Paper server | `JavaPlugin`, `plugin.yml`, `paper-api` dependency, events, commands, a `plugins/` folder | [plugin-dev/OVERVIEW.md](plugin-dev/OVERVIEW.md) |
| **Contributing to Paper** — changing Paper *itself* | Patch files, `./gradlew applyPatches`, `paper-server/`, NMS/mappings, opening a PR on `PaperMC/Paper` | **Not built yet** — see below |

If it is ambiguous, ask. "Add an event" means one thing in a plugin and something completely
different inside Paper's own source tree.

Whichever prong applies: **do not answer from memory.** Paper moves fast, and recalled Bukkit-era
patterns are the main failure mode here. Each prong names its own ground truth and how to fetch
it.

## Contributing to Paper — not yet available

The second prong is not written. Do **not** substitute plugin-development guidance for it: the
plugin API is not how you change Paper's internals, and answering from the plugin side would be
actively misleading.

Say so plainly, then offer what genuinely helps:

- `PaperMC/Paper`'s own `CONTRIBUTING.md` is the authoritative source, and Paper uses a
  patch-based workflow against upstream — contributions are patch files, not direct edits to
  vendored source.
- The docs site is **not** the ground truth for this prong. It documents plugin development and
  server administration; it carries exactly one contributor-facing page,
  `paper/contributing/events.md` (adding an event to Paper). If the plugin-dev prong has already
  cloned the docs to `.claude/extern/papermc-docs/`, that page is there — otherwise it alone is
  not worth cloning for.
- `paper/dev/misc/internal-code.md` and `paper/dev/getting-started/userdev.md` cover Minecraft
  internals and `paperweight-userdev`, which sit closest to contributor territory.
