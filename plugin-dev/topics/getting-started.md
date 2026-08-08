# Topic: getting started

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/getting-started/project-setup.mdx` | 244 | — | Creating a new plugin project: Gradle Kotlin DSL, `paper-api` dependency, `src` layout, main class, Java toolchain |
| `paper/dev/getting-started/plugin-yml.mdx` | 250 | — | Writing `plugin.yml`: name, main, `api-version`, dependencies, permissions, command declarations |
| `paper/dev/getting-started/how-do-plugins-work.md` | 125 | — | Explaining the plugin model or lifecycle to the user; the constructor/`onLoad`/`onEnable` caveats |
| `paper/dev/getting-started/paper-plugins.md` | 232 | — | Only when the user wants `paper-plugin.yml`, a bootstrapper, or a loader ⚠ badged Experimental |
| `paper/dev/getting-started/userdev.md` | 189 | — | The user needs `paperweight-userdev` to touch Minecraft internals / NMS |
| `misc/java-install.md` | 187 | — | The user is on the wrong JDK. Current docs target **Java 25** |
| `paper/admin/reference/paper-plugins.md` | 45 | — | Server-side view of how Paper plugins load — useful when diagnosing load-order problems |

## Notes

- **Build system:** Gradle Kotlin DSL. The docs label Maven "Discouraged" and only show it in a
  secondary tab.
- **Never hardcode the version.** `project-setup.mdx` shows `\{LATEST_PAPER_RELEASE}` — resolve
  it per `../conventions.md` §1.
- **Testing the plugin:** `project-setup.mdx` closes by recommending the
  [Run-Task](https://github.com/jpenilla/run-task) Gradle plugin, which downloads and runs a
  Paper server. Good default when the user asks how to test.
- Output JAR lands in `build/libs`.

## Do not read

`paper/dev/index.mdx`, `paper/dev/getting-started/index.mdx` — 11-line sidebar stubs, no content.
