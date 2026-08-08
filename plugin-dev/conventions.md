# Reading the PaperMC docs correctly

The docs are Astro/Starlight source, not plain Markdown. Four things will produce broken output
if you read them literally.

## 1. Version placeholders are literal in the file

`.mdx` code blocks tagged `replace` contain build-time placeholders. A raw file read shows the
placeholder, not a version:

```kotlin
compileOnly("io.papermc.paper:paper-api:\{LATEST_PAPER_RELEASE}.build.+")
```

The ones you will hit: `\{LATEST_PAPER_RELEASE}`, `\{LATEST_PAPER_BUILD_API_VERSION}`,
`\{LATEST_VELOCITY_RELEASE}`, `\{LATEST_FOLIA_RELEASE}`, `\{LATEST_USERDEV_RELEASE}`,
`\{LATEST_MC_RELEASE}`, `\{LATEST_ADVENTURE_API_RELEASE}`.

**Never copy a placeholder into a real file, and never guess the version.** The docs resolve
these from the live Paper API at build time; do the same.

### Identify yourself — the fill API requires it

PaperMC requires every request to `fill.papermc.io` to carry a User-Agent that names the
software and gives a contact URL or email; generic agents (bare `curl`, `wget`) do not qualify.
Enforcement is not yet switched on — a bare `curl` still returns 200 today — so treat this as a
stated policy that can start being enforced without notice, and comply regardless. Always export
this first and pass it on every call:

```bash
UA="marginalia-mc/1.0 (+https://claude.com/claude-code)"
```

If the user has their own project identity or contact, prefer theirs — the requirement is that
the header identifies a real caller, not that it says marginalia.

### Resolving versions and builds

```bash
UA="marginalia-mc/1.0 (+https://claude.com/claude-code)"

# newest Minecraft version Paper publishes for
curl -s -H "User-Agent: $UA" https://fill.papermc.io/v3/projects/paper \
  | jq -r '.versions | to_entries[0] | .value[0]'

# newest STABLE build for a version ("null" means none yet -- fall back a version)
curl -s -H "User-Agent: $UA" https://fill.papermc.io/v3/projects/paper/versions/<version>/builds \
  | jq -r 'map(select(.channel == "STABLE")) | .[0] | .id'

# all builds with channels, newest first -- use when no STABLE build exists
curl -s -H "User-Agent: $UA" https://fill.papermc.io/v3/projects/paper/versions/<version>/builds \
  | jq -r '.[] | "\(.id) \(.channel)"' | head
```

Substitute the same way the docs do — `LATEST_PAPER_RELEASE` is the newest version that has at
least one non-`ALPHA` build, not simply the newest version. Full API reference:
`misc/downloads-service.mdx`.

**Dependency coordinate:**

```kotlin
compileOnly("io.papermc.paper:paper-api:<version>.build.+")   // Gradle, floating latest build
```

Maven needs a range: `[<version>.build,)`. Pin a specific build by replacing `+` with e.g.
`25-alpha`.

**Version scheme changed.** `1.21.11` and below used `{VERSION}-R0.1-SNAPSHOT`, with no way to
reference a build. Newer releases use CalVer (`26.1`, `26.2`) with `.build.N-<channel>`. Check
which era you are targeting before writing the coordinate.

Java toolchain in the current docs is **25**.

## 2. `jd:` links are docs-only syntax

```
[`JavaPlugin`](jd:paper:org.bukkit.plugin.java.JavaPlugin)
[`spawnParticle`](jd:paper:org.bukkit.World#spawnParticle(org.bukkit.Particle,double,double,double,int))
```

Format is `jd:<javadoc>[:<module>]:<fully.qualified.Class>` — `.` separates packages, `$` marks
an inner class, `#` a member. It renders to a `https://jd.papermc.io/...` URL.

It is a **link macro**. It must never appear in generated Java, in comments, or in anything you
show the user as code. Read it as "the class is `org.bukkit.plugin.java.JavaPlugin`" and use the
plain class name.

## 3. Cross-links use URL slugs, and no slug matches its file path

Docs link to each other by slug (`/paper/dev/scheduler`), which is set in frontmatter and is
almost never the file path:

| Link in a doc | Actual file |
|---|---|
| `/paper/dev/scheduler` | `paper/dev/api/scheduler.md` |
| `/paper/dev/chat-events` | `paper/dev/api/event-api/chat-event.md` |
| `/paper/dev/component-api/introduction` | `paper/dev/api/component-api/intro.md` |
| `/paper/dev/display-entities` | `paper/dev/api/entity-api/display-entities.md` |
| `/paper/dev/project-setup` | `paper/dev/getting-started/project-setup.mdx` |

The transformation is irregular — `/api/` is dropped, `entity-api/` and `event-api/` flatten
away, and some basenames are renamed (`chat-event` → `chat-events`). Do not try to derive it.

**Resolve a slug to a file** — exact, and never goes stale:

```bash
grep -rl '^slug: paper/dev/scheduler$' .claude/extern/papermc-docs/src/content/docs/
```

Thirteen files (mostly `index.mdx` sidebar stubs) have no `slug:` and fall back to their path.

## 4. MDX components carry real content

- **`<Tabs syncKey="build-system">` / `<TabItem label="...">`** — usually Gradle vs Maven. Take
  the **Gradle Kotlin DSL** branch; the docs label Maven "Discouraged". Also used for
  `ParticleBuilder` vs `spawnParticle` — prefer `ParticleBuilder`.
- **`<FileTree>`** — directory layouts.
- **`:::note` / `:::tip` / `:::caution` / `:::danger`** — these hold the load-bearing warnings
  (chat-signature restrictions on `run_command`, constructor caveats, thread-safety). Do not
  skim past them.
- **`<Badge text="Experimental">`, `sidebar.badge`** — a weak signal. Paper applies it liberally
  and removes it rarely, so it marks a page as *subject to change*, not as off-limits. Your
  prong's guidance says how to act on it.
- **`import` lines and `<Image .../>`** — noise. Screenshots of chat text carry nothing you can
  use.
- **`version:` frontmatter** — minimum Minecraft/Paper version for that API. Fifteen docs have
  one; `topics/` tables carry it. If the user targets older, that API does not exist for them.
