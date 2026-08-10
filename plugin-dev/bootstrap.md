# Bootstrap: getting and refreshing the PaperMC docs

The skill reads the real PaperMC docs from a clone inside the project. This file covers
getting that clone, keeping it fresh, and what to do when upstream moves files.

Run the check below **once per session**, before the first doc lookup.

## Plan mode blocks this — bootstrap before you plan

Every step here writes: `git clone`, `git pull`, appending to `.gitignore`. Plan mode is
read-only, so a planning agent cannot bootstrap — it silently skips setup and plans a Paper task
from memory, the one failure this skill exists to prevent. The clone is therefore a
**prerequisite to planning, not a step inside the plan**:

- **Clone already present** → plan mode is fine. Reading the docs is read-only. The staleness
  refresh (step 2) is the only write, and a slightly stale clone still beats memory — note it and
  carry on, refresh once you are out of plan mode.
- **Clone missing** → stop; do not draft a plan yet. Leave plan mode, run step 1 to clone, then
  re-enter planning with the docs in hand. Cloning is a one-time, ~2 MB, gitignored setup that
  never touches the user's own tree, so it is safe to do before the plan is approved.

## 1. Is the clone present?

```bash
test -d .claude/extern/papermc-docs/.git && echo present || echo missing
```

**Missing → clone it.** Run from the project root:

```bash
mkdir -p .claude/extern
git clone --filter=blob:none --sparse \
  https://github.com/PaperMC/docs.git .claude/extern/papermc-docs
git -C .claude/extern/papermc-docs sparse-checkout set --no-cone \
  'src/content/docs/**' '!src/content/docs/**/assets/**'
```

This lands ~2 MB instead of ~172 MB. The excluded files are screenshots, GIFs, and MP4s that
are useless to an agent. It is still a normal git repo — `git pull` works.

> If you ever need the images (you generally do not), drop the second command and run
> `git -C .claude/extern/papermc-docs sparse-checkout disable`.

Then keep it out of the user's history — it is a vendored third-party repo under BSD 2-Clause
(site code) plus CC-BY-SA-4.0 (`LICENSE-docs`, the documentation content):

```bash
grep -qxF '.claude/extern/' .gitignore 2>/dev/null || echo '.claude/extern/' >> .gitignore
```

Tell the user once that you cloned the docs and added the ignore entry.

## 2. Is it stale? (30-day refresh)

`.git/FETCH_HEAD` is written on every fetch, so its mtime *is* the last-fetch time — no extra
state file needed. A fresh clone may not have it; fall back to `.git`.

```bash
D=.claude/extern/papermc-docs
REF=$D/.git/FETCH_HEAD; [ -f "$REF" ] || REF=$D/.git
AGE=$(( ( $(date +%s) - $(stat -c %Y "$REF") ) / 86400 ))
echo "docs last fetched ${AGE}d ago"
[ "$AGE" -gt 30 ] && echo STALE || echo FRESH
```

**Stale → refresh, then always run step 3:**

```bash
git -C .claude/extern/papermc-docs pull --ff-only
```

If the pull fails (rebased upstream, dirty sparse state), delete
`.claude/extern/papermc-docs` and re-clone from step 1. Do not fight it.

## 3. Drift check — did upstream move anything?

Upstream reorganises. This is not hypothetical: every one of the docs' URL slugs already
diverges from its file path, so this repo demonstrably shuffles files. When it does, the paths
in `topics/` go stale and point at nothing.

After every pull, check that the paths this skill indexes still exist:

```bash
grep -rhoE '`(paper|adventure|misc|folia)/[A-Za-z0-9/_.-]+\.mdx?`' <skill-dir>/plugin-dev/topics \
  | tr -d '`' | sort -u \
  | while read -r p; do
      [ -f ".claude/extern/papermc-docs/src/content/docs/$p" ] || echo "MISSING $p"
    done
```

Silence means clean. **If it prints anything, stop and tell the user**, with the paths it named:

> The PaperMC docs moved N file(s) that this skill indexes: `<old paths>`. The index needs
> updating — please open an issue or PR on the marginalia repo. I can still work, but I'll be
> reading around the gap.

Then find the current location (`git -C .claude/extern/papermc-docs log --diff-filter=R
--name-status -1 -- '*<basename>*'`, or just search the tree) and use it for this session.

**Do not** silently substitute a guessed path, and **do not** fall back to answering from
memory. A wrong path is recoverable; a confidently wrong API is not.

## 4. Digest staleness

`digests/` holds condensed copies of five upstream files. Each digest header records the blob SHA
it was written from. Blob SHAs change if and only if content changes, so this is exact:

```bash
for f in adventure/minimessage/format.mdx adventure/text.md adventure/audiences.md \
         adventure/minimessage/api.mdx adventure/minimessage/dynamic-replacements.md; do
  echo "$(git -C .claude/extern/papermc-docs rev-parse HEAD:src/content/docs/$f)  $f"
done
```

Compare each against the SHAs recorded in `digests/*.md`. Any mismatch → **that digest may be
behind**. Tell the user which one, and read the upstream file rather than trusting the digest for
anything it does not clearly cover.

This matters more than the path check: a moved path fails loudly on the next read, but a stale
digest keeps returning confident, outdated answers.
