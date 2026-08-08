# Topic: lifecycle, registries, and internals

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/lifecycle/lifecycle.md` | 154 | — | Any `LifecycleEvents` work — **command registration goes through this** |
| `paper/dev/api/registries.md` | 174 | — | Reading or modifying registries (enchantments, items, biomes…) ⚠ badged Experimental |
| `paper/dev/api/lifecycle/datapacks.mdx` | 178 | 1.21.4+ | Shipping a datapack inside your plugin JAR ⚠ badged Experimental |
| `paper/dev/misc/internal-code.md` | 109 | — | The user insists on NMS / Minecraft internals; the caveats |
| `paper/dev/getting-started/userdev.md` | 189 | — | Setting up `paperweight-userdev` to compile against internals |

## Notes

- **The Lifecycle API is the modern registration mechanism.** Commands, and increasingly other
  registrations, hang off it rather than `onEnable` calls. If a doc says "register in
  `onEnable`" and another says lifecycle, lifecycle is the newer one.
- **Registry modification happens early** — often in a bootstrapper, before the server exists.
  That is one of the few genuine reasons to use `paper-plugin.yml`
  ([getting-started.md](getting-started.md)).
- **On the Experimental badges here:** both `registries.md` and `datapacks.mdx` carry one. Per
  the note in [../OVERVIEW.md](../OVERVIEW.md), that means the signature may shift, not that the
  API is unusable. Mention it once and proceed.
- **Internals are a last resort.** `internal-code.md` is blunt about the cost: obfuscation
  mappings, breakage every Minecraft update, and no API stability. Exhaust the API first, and
  say so to the user before reaching for NMS.

## Do not read

`paper/dev/api/lifecycle/index.mdx`, `paper/dev/api/index.mdx` — 8–11 line stubs.
`paper/dev/api/roadmap.md` (94) — project intentions, not usable API.
