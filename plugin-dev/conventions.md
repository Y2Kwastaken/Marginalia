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

### Identify yourself on every request

Set an identifying User-Agent on **every HTTP request you make while working** — not only
`fill.papermc.io`, but any curl at all: Hangar, the javadoc host, a downloads mirror, an API you
hit to check something mid-build. Export it once and pass it on every call:

```bash
UA="marginalia-mc/1.0 (+https://github.com/Y2Kwastaken/Marginalia)"
curl -s -H "User-Agent: $UA" https://...
```

PaperMC *requires* it specifically — `fill.papermc.io` rejects a User-Agent that does not name
the software with a contact URL or email (bare `curl`, `wget` do not qualify). Enforcement is not
switched on yet — a bare `curl` still returns 200 today — so treat it as a stated policy that can
start being enforced without notice. But the rule here is broader than that one host: identifying
the caller on every request is this skill's default, PaperMC or not.

Two carve-outs:

- **Scripts the user ships.** When you write a curl into code the *user* will run, use the user's
  own project identity/contact, not marginalia — the header must name the real caller. The
  marginalia UA is for requests *you* make on their behalf during the work.
- If the user has given you their own identity for your requests too, prefer theirs — the
  requirement is a real caller, not the literal string marginalia.

### Resolving versions and builds

```bash
UA="marginalia-mc/1.0 (+https://github.com/Y2Kwastaken/Marginalia)"

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

## 5. Probe the compiled API with `javap`, never `jar -xf`

The docs are prose — they do not carry every signature, overload, or enum constant. When you need
the exact shape of an API and the docs do not give it, read it from the jar the project already
compiled against. A `compileOnly("io.papermc.paper:paper-api:…")` dependency means gradle has
**already** downloaded that jar into the cache; read it in place:

```bash
JAR=$(find ~/.gradle/caches -name 'paper-api-*.jar' \
        ! -name '*-sources.jar' ! -name '*-javadoc.jar' | head -1)
javap -p -cp "$JAR" org.bukkit.World
```

**Never `jar -xf` the jar.** It explodes thousands of `.class` files — JVM bytecode, not readable
source — into the working directory, and you would have to run `javap` on the result anyway.
`javap` reads straight out of the jar, writes nothing to disk, and takes the class(es) you name.
The `-p` flag is load-bearing: without it you only see `public` members and miss protected hooks.

The cached jar **is** the exact build the project links against, so its signatures are
authoritative for this project — no version-matching risk. If several versions are cached, pick
the one the project resolved rather than `head -1`. Do not download a separate jar just to check a
signature.

**When `javap` is not enough** — you need the javadoc prose or real parameter names (`javap`
prints `arg0`, `arg1`) — read the matching `-sources.jar` instead, pinned to the *same* build:

```bash
unzip -p "$(find ~/.gradle/caches -name 'paper-api-*-sources.jar' | head -1)" \
  org/bukkit/World.java
```

The sources jar is not fetched by default; it is present only if the build (or an IDE sync) asked
for it. Fall back to `javap` on the main jar, or the `jd:` javadoc (§2), when it is absent.
