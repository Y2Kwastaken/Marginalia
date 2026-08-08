# Using marginalia

For agents and tools that want to **consume** this skill while working on a Minecraft plugin.

If you are here to **change** marginalia itself — add an index entry, update a digest, fix a
path — you want [AGENTS.md](../AGENTS.md) instead.

## What you are getting

A map to the PaperMC documentation, not a copy of it. marginalia tells you which file to open
for a given task and what will bite you when you read it. The documentation itself gets cloned
at runtime.

The chain is always: `SKILL.md` → a prong → one topic table → one or two upstream doc files.
Follow it in that order and stop as soon as you have what you need.

## If your tool supports Claude Code skills

Nothing to do beyond installing it. `SKILL.md` carries frontmatter, and the skill activates on
its own when a task looks like Paper work.

```bash
ln -s /path/to/marginalia ~/.claude/skills/marginalia
```

## If your tool does not (Codex, Cursor, Gemini CLI, Copilot, …)

These read a project instruction file — `AGENTS.md`, `.cursor/rules`, `GEMINI.md`,
`.github/copilot-instructions.md` — and have no equivalent of skill auto-discovery. Nothing here
will load by itself. Wire it up in two steps.

**1. Vendor marginalia into the plugin project.**

```bash
git submodule add <marginalia-repo-url> .agents/marginalia
# or, if you do not want a submodule:
git clone <marginalia-repo-url> .agents/marginalia
```

**2. Point your project's instruction file at it.** Add something like this to the plugin
project's `AGENTS.md` (or equivalent):

```markdown
## PaperMC work

This project is a Paper plugin. Before writing or changing any Paper/Bukkit/Spigot code,
read `.agents/marginalia/SKILL.md` and follow its routing. Do not answer Paper API
questions from memory — the API moves fast and recalled Bukkit-era patterns are wrong.
```

That single pointer is enough; everything downstream works unchanged, because the rest of
marginalia is ordinary Markdown with relative links.

## The one setup step

Before your first lookup, follow [plugin-dev/bootstrap.md](../plugin-dev/bootstrap.md). It clones
the PaperMC docs into `.claude/extern/papermc-docs/` (~2 MB, markdown only), refreshes them if
the last fetch was over 30 days ago, and tells you what to do if upstream moved a file.

**On that path:** `.claude/extern/` is just a directory name, not a Claude requirement — any
tool can read it, and keeping it means the paths in `topics/` work as written. If you must put
the checkout somewhere else, every path in this skill is relative to the checkout's
`src/content/docs/`, so substitute that prefix consistently.

## Rules while using it

- **Do not guess a path.** If an indexed file is not where the table says, upstream moved it.
  Say so and report it to the marginalia repository. A wrong path is recoverable; a confidently
  wrong API is not.
- **Do not edit the index to make a lookup succeed.** Using and maintaining are separate jobs.
  If something is wrong, report it rather than patching it mid-task.
- **Read [plugin-dev/conventions.md](../plugin-dev/conventions.md) once per session** before opening
  any doc file. Version placeholders are literal in the source, `jd:` links are not Java, and
  cross-links use slugs that never match file paths. Skipping it produces broken output.
- **Prefer the digests** in `plugin-dev/digests/` over their upstream sources — they are
  condensed for exactly this purpose. Each records the blob SHA it was built from; if that no
  longer matches upstream, treat the digest as possibly stale and read the source.
- **Open one topic file, not all of them.** The whole design is to keep the rest out of context.

## What is not here

Only the plugin-development prong is built. Contributing patches to Paper itself — the patch
workflow, `paper-server/`, mappings — is not covered. If that is your task, `SKILL.md` will tell
you to stop and go to `PaperMC/Paper`'s own `CONTRIBUTING.md`. Do not let plugin API guidance
stand in for it; they are different problems.
