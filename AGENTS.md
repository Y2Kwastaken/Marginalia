# AGENTS.md

Guidance for any coding agent **working on this repository** — editing the index, updating a
digest, fixing a path.

> **Just want to use marginalia to write a Paper plugin?** This is the wrong file. Read
> [USING.md](docs/USING.md), then [SKILL.md](SKILL.md). Nothing below applies to you.

## What this repo is

`marginalia` is a **Claude Code skill**, not an application. There is no build, no test suite, and
no runtime — it is Markdown that instructs an agent where to look in the PaperMC documentation
and what to watch out for.

```
SKILL.md              routes: plugin work, or Paper-core work?
docs/USING.md         consumer-facing counterpart to this file
plugin-dev/
├── OVERVIEW.md       prong entry: setup + core rules + topic router
├── bootstrap.md      clone/refresh/drift for the docs checkout
├── conventions.md    how to read Astro/Starlight docs safely
├── topics/*.md       lookup tables: path, line count, min version, "read when"
├── topics/_skip.txt  deliberately unindexed docs, with rationale
└── digests/*.md      condensed adaptations of upstream docs
```

`SKILL.md` is the only file with skill frontmatter. The prong entry is `OVERVIEW.md`, **not**
`SKILL.md`, so skill discovery does not pick up a second skill one level down. Keep it that way.

**Root is reserved for discovery.** `SKILL.md` (Claude Code skills), `README.md` (GitHub),
`AGENTS.md` (Codex and similar), and `CLAUDE.md` are each found by name at the repo root and
cannot move. Anything not pinned that way belongs in a subfolder — new maintainer or user
documentation goes in `docs/`.

## The one rule that matters

**Never write a doc path, line count, version, or API claim you have not verified against a real
checkout.** This entire skill is a set of factual assertions about another repository. An invented
path is worse than no entry, because the agent trusts it and then flails.

Get a checkout to verify against:

```bash
export REPO=/tmp/pmc-docs
git clone --filter=blob:none --sparse https://github.com/PaperMC/docs.git "$REPO"
git -C "$REPO" sparse-checkout set --no-cone \
  'src/content/docs/**' '!src/content/docs/**/assets/**'
export DOCS="$REPO/src/content/docs"
```

Line counts come from `wc -l`, descriptions and `version:` minimums come from each file's
frontmatter — not from memory or inference.

## Checks before you commit

Run all four from the repo root with `$REPO` and `$DOCS` set as above.

**1. Every indexed path exists.**

```bash
grep -rhoE '`(paper|adventure|misc|folia)/[A-Za-z0-9/_.-]+\.mdx?`' plugin-dev/topics \
  | tr -d '`' | sort -u \
  | while read -r p; do [ -f "$DOCS/$p" ] || echo "MISSING $p"; done
```

**2. Coverage — no upstream page silently unindexed.**

```bash
find "$DOCS/paper/dev" "$DOCS/adventure" -name '*.md' -o -name '*.mdx' \
  | sed "s|^$DOCS/||" | grep -v '/index\.mdx$' | sort \
  | while read -r p; do
      grep -qxF "$p" plugin-dev/topics/_skip.txt && continue
      grep -rqF "$p" plugin-dev/topics || echo "unindexed $p"
    done
```

New upstream page → index it in the right topic file, or add it to `_skip.txt` **with a reason**.

**3. Digest sources unchanged.** Each digest header records the blob SHA it was written from;
blob SHAs change iff content changes.

```bash
for f in adventure/minimessage/format.mdx adventure/text.md adventure/audiences.md \
         adventure/minimessage/api.mdx adventure/minimessage/dynamic-replacements.md; do
  sha=$(git -C "$REPO" rev-parse "HEAD:src/content/docs/$f")
  grep -qF "$sha" plugin-dev/digests/*.md || echo "DRIFTED $f -> $sha"
done
```

Drifted → re-read the upstream file, update the digest, update the recorded SHA.

**4. Internal links resolve.**

```bash
while IFS=: read -r src link; do
  [ -f "$(dirname "$src")/$link" ] || echo "BROKEN $src -> $link"
done < <(grep -rnoE '\]\(([A-Za-z0-9_./-]+\.(md|txt))\)' . --include=*.md \
         | sed -E 's/:[0-9]+:\]\(/:/; s/\)$//')
```

## Conventions

- **Relative links.** `topics/` and `digests/` reach siblings by bare filename and the prong root
  with `../`. Do not write `topics/foo.md` from inside `topics/`.
- **Keep `SKILL.md` and `OVERVIEW.md` small.** They load every time. Detail belongs in a topic
  file that loads only when relevant.
- **Index, do not summarise.** Condense a doc only when it is mostly scaffolding — repeated
  headings, screenshots, boilerplate. Dense reference (`particles.mdx`, the command API) gets
  section anchors instead, because summarising it drops facts. Any new digest must record its
  source paths and blob SHAs.
- **No scripts.** This was deliberate: a committed shell script here validated its own formatting
  assumptions and could report success without having run. Checks live inline where they are used.
- **`fill.papermc.io` requires a User-Agent** naming the software with a contact URL. Use
  `marginalia-mc/1.0 (+https://claude.com/claude-code)`.

## Licensing constraint

The files in `plugin-dev/digests/` are condensed adaptations of **CC-BY-SA-4.0** documentation,
which makes them derivative works — attribution and ShareAlike genuinely apply. Keep the source
headers in each digest and the credits section in `README.md`. This also constrains what licence
this repository can carry; do not add a conflicting one without checking.

## Status

`plugin-dev/` is built. The second prong — contributing patches to `PaperMC/Paper` — is not.
Do not stub it or let plugin-development guidance answer Paper-core questions; `SKILL.md`
deliberately instructs the agent to decline and point at `PaperMC/Paper`'s `CONTRIBUTING.md`.
The docs site is **not** the ground truth for that prong.
