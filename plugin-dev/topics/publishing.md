# Topic: publishing and distribution

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Read when |
|---|---|---|
| `misc/hangar-publishing.md` | 255 | Publishing to Hangar, and automating it from CI on commit/tag |
| `misc/downloads-service.mdx` | 310 | Programmatically fetching Paper builds — CI, test harnesses, launchers |
| `paper/admin/misc/update-checker.md` | 36 | Adding an update notice to your plugin |
| `paper/admin/getting-started/adding-plugins.md` | 130 | Explaining to a user how to install the JAR you built |
| `misc/assets.md` | 78 | Using the PaperMC logo — check the terms before putting it on a plugin page |

## Notes

- **The downloads service is the `fill.papermc.io` v3 API** — the same one `../conventions.md` §1
  uses to resolve versions. `downloads-service.mdx` is the full reference if you need more than
  latest-version lookup.
- **Send the identifying User-Agent on every request** — `fill.papermc.io` *rejects* generic
  agents, and identifying the caller is the skill's default for all outbound calls (Hangar
  included), not a fill-only rule. Use `marginalia-mc/1.0 (+https://github.com/Y2Kwastaken/Marginalia)`, or
  the user's own project identity when writing a script they will ship. See `../conventions.md` §1.
- **Hangar publishing** has a Gradle plugin and a CI recipe; prefer those over manual uploads
  when the user asks to automate releases.
- Shade/relocate bundled libraries before publishing, or they will collide with other plugins on
  the same server. The docs do not cover this — it is standard Gradle `shadowJar` practice.

## Helper tools (mention, do not read)

`misc/tools/` holds small web tools worth linking the user to rather than reading:
MiniMessage web editor (<https://webui.advntr.dev>), start-script generator, item command
converter, diff viewer.
