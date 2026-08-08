# Topic: debugging and diagnosis

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/misc/reading-stacktraces.md` | 67 | — | The user pasted a stacktrace. Short — read it |
| `paper/dev/misc/debugging.md` | 92 | — | Attaching a remote debugger, breakpoints, IDE run config |
| `paper/admin/how-to/basic-troubleshooting.mdx` | 233 | — | Server-side symptoms: won't start, crashes, plugin conflicts |
| `paper/admin/how-to/profiling.md` | 53 | — | The complaint is lag/TPS. Spark profiling |
| `paper/dev/misc/internal-code.md` | 109 | — | The stacktrace runs into obfuscated internals |

## Server configuration reference

Useful when the bug turns out to be config, not code.

| Path | Lines | Read when |
|---|---|---|
| `paper/admin/reference/configuration/index.mdx` | 136 | Which config file owns a setting |
| `paper/admin/reference/system-properties.md` | 335 | JVM flags and env vars Paper reads |
| `paper/admin/reference/cli-arguments.md` | 130 | Server launch flags (min ver 1.21.11) |
| `paper/admin/reference/commands.md` | 218 | Paper's own commands (min ver 1.21.8) |
| `paper/admin/misc/paper-bug-fixes.md` | 74 | Behaviour differs from vanilla and it may be deliberate |
| `paper/admin/how-to/aikars-flags.md` | 175 | JVM/GC tuning for the test or production server |

## Notes

- **Read the stacktrace before theorising.** The plugin frame nearest the top is usually yours;
  `reading-stacktraces.md` covers `Caused by:` chains, which is where the real fault normally is.
- **"Plugin X generated an exception"** names the plugin the listener belongs to — that is who
  threw, not necessarily who is at fault.
- **`NoSuchMethodError` / `NoClassDefFoundError` at runtime** almost always means compiled
  against a different `paper-api` than the server is running. Check the version before anything
  else (`../conventions.md` §1).
- **Lag is measured, not guessed.** Point the user at Spark rather than speculating about which
  listener is slow.
- Paper deliberately fixes vanilla bugs; if behaviour differs from vanilla, check
  `paper-bug-fixes.md` before treating it as your plugin's fault.
