# marginalia

A Claude Code skill for [PaperMC](https://papermc.io) work, split into two prongs:

| Prong | Status | Purpose |
|---|---|---|
| `plugin-dev/` | **Built** | Writing, building, and debugging Paper server plugins |
| Contributing to Paper | Not yet built | Patches and PRs against `PaperMC/Paper` itself |

## How it works

The skill ships **no documentation content** — it ships a *map*.

On first use it clones [PaperMC/docs](https://github.com/PaperMC/docs) into
`.claude/extern/papermc-docs/` (a ~2 MB sparse, markdown-only checkout), then opens only the
two or three files a given task needs. `SKILL.md` routes to a prong, the prong routes to one
small topic table, and nothing else enters context.

```
marginalia/
├── SKILL.md              routes: plugin work, or Paper-core work?
├── AGENTS.md             for agents editing this repo
├── docs/USING.md         for agents using the skill
└── plugin-dev/           the built prong
    ├── OVERVIEW.md       setup + core rules + topic router
    ├── bootstrap.md      clone, 30-day refresh, drift handling
    ├── conventions.md    reading Astro/Starlight docs without getting burned
    ├── topics/           one lookup table per task area
    └── digests/          two condensed references
```

Only `SKILL.md`, `README.md`, `AGENTS.md`, and `CLAUDE.md` sit at the root, and each is pinned
there by a tool that looks for it by name. Everything else lives in a subfolder.

Paths and reading strategy age far more slowly than API details, which is why the skill indexes
rather than embeds. When upstream does move a file, the skill is instructed to say so and ask
for a fix here — never to guess a replacement path or fall back to memory.

## Install

```bash
ln -s "$PWD" ~/.claude/skills/marginalia
```

Using a tool that does not support Claude Code skills — Codex, Cursor, Gemini CLI, Copilot?
The content is plain Markdown and works fine; only the auto-discovery is Claude-specific. See
[USING.md](docs/USING.md) for the two-step wiring.

| You want to… | Read |
|---|---|
| **Use** marginalia to write a plugin | [USING.md](docs/USING.md) → [SKILL.md](SKILL.md) |
| **Edit** marginalia — index entries, digests, paths | [AGENTS.md](AGENTS.md) |

## Credits

An initial burst from me here is please contribute to PaperMC docs. The work of the docs team and coverage directly
benefits this skill. All pull requests to that repository should be as high quality as possible. In a way this repository
acts as a parasite. I just want to make sure to share the love here and tell anyone reading this go contribute!

Everything useful in this skill comes from the people who wrote the PaperMC documentation. The
repository has many contributors. I personally send thanks to them all, and in particular to the top
contributors below (`PaperMC/docs`, fetched 2026-08-08):

| | | |
|---|---|---|
| [zlataovce](https://github.com/zlataovce) — 159 | [olijeffers0n](https://github.com/olijeffers0n) — 112 | [e-im](https://github.com/e-im) — 98 |
| [Strokkur424](https://github.com/Strokkur424) — 77 | [kashike](https://github.com/kashike) — 33 | [Doc94](https://github.com/Doc94) — 32 |
| [powercasgamer](https://github.com/powercasgamer) — 27 | [Machine-Maker](https://github.com/Machine-Maker) — 26 | [4drian3d](https://github.com/4drian3d) — 22 |
| [Lulu13022002](https://github.com/Lulu13022002) — 20 | [lynxplay](https://github.com/lynxplay) — 20 | [Warriorrrr](https://github.com/Warriorrrr) — 17 |
| [kennytv](https://github.com/kennytv) — 15 | [Leguan16](https://github.com/Leguan16) — 15 | [MiniDigger](https://github.com/MiniDigger) — 14 |
| [Timongcraft](https://github.com/Timongcraft) — 14 | [NoahvdAa](https://github.com/NoahvdAa) — 13 | [456dev](https://github.com/456dev) — 13 |
| [Owen1212055](https://github.com/Owen1212055) — 10 | [Nacioszeczek](https://github.com/Nacioszeczek) — 10 | |

Full list: [github.com/PaperMC/docs/graphs/contributors](https://github.com/PaperMC/docs/graphs/contributors).

Thank you also to the [Adventure](https://github.com/KyoriPowered/adventure) authors — the
`adventure/` section of the docs, and both digests here, are built on that work.

## Affiliation

**Not affiliated with, sponsored by, or endorsed by PaperMC.** This is an independently owned
skill that indexes their publicly available documentation for use by coding agents.

Please do not report issues with this skill to PaperMC, that organization is **NOT** responsible for this skill 
and did not build it. All issues belong in this repository. Anything raised with PaperMC first will be
closed here without consideration. I have absolutely 0 tolerance for such behaviors.
