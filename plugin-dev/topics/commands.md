# Topic: commands (Brigadier)

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

**Default to Brigadier.** `CommandExecutor` / `onCommand` / `plugin.yml` `commands:` is the
legacy path — use it only when editing code that already does.

## Basics — read these first

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/command-api/basics/introduction.md` | 55 | — | Starting any command work. Short — read it |
| `paper/dev/api/command-api/basics/command-tree.mdx` | 290 | — | Structuring literals/arguments, nested subcommands |
| `paper/dev/api/command-api/basics/arguments-and-literals.md` | 176 | — | What an argument vs a literal is; the `ArgumentType` set |
| `paper/dev/api/command-api/basics/executors.md` | 152 | — | Writing the code that runs; `Command`/`CommandContext`, return values |
| `paper/dev/api/command-api/basics/registration.md` | 133 | — | Wiring the command up via the Lifecycle API — the part people get wrong |
| `paper/dev/api/command-api/basics/requirements.md` | 114 | 1.21.6 | Permission-gating; hiding a command from players who cannot use it |
| `paper/dev/api/command-api/basics/argument-suggestions.mdx` | 201 | — | Tab completion, custom suggestion providers, tooltips |
| `paper/dev/api/command-api/basics/custom-arguments.md` | 287 | — | An argument type Brigadier does not provide |

## Argument types — open only the one you need

| Path | Lines | Min ver | Covers |
|---|---|---|---|
| `paper/dev/api/command-api/arguments/minecraft.md` | 44 | — | The essential Brigadier argument types |
| `paper/dev/api/command-api/arguments/location.mdx` | 99 | — | `BlockPosition`, `FinePosition`, `World` |
| `paper/dev/api/command-api/arguments/entity-player.mdx` | 186 | — | Player and entity selectors, player profiles |
| `paper/dev/api/command-api/arguments/registry.mdx` | 166 | 26.1.1 | Registry values — enchantments, items, `ResourceKey` |
| `paper/dev/api/command-api/arguments/paper.mdx` | 186 | — | Paper-specific values (`NamespacedKey`, heightmap, …) |
| `paper/dev/api/command-api/arguments/enums.mdx` | 168 | — | `GameMode`, `EntityAnchor`, similar enums |
| `paper/dev/api/command-api/arguments/predicate.mdx` | 102 | — | Item/block predicates, validation |
| `paper/dev/api/command-api/arguments/adventure.mdx` | 190 | — | Arguments returning Adventure objects (`Component`, `Style`, `NamedTextColor`) |

## Migration and comparison

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/command-api/misc/comparison-bukkit-brigadier.md` | 110 | — | Porting an existing `CommandExecutor` to Brigadier |
| `paper/dev/api/command-api/misc/basic-command.md` | 318 | 1.21.1 | The user wants a simple Bukkit-shaped command without a full Brigadier tree (`BasicCommand`) |

## Notes

- **Registration is lifecycle-based**, not `getCommand()`. See `registration.md`; it uses
  `LifecycleEvents.COMMANDS`. Cross-reference [lifecycle-registries.md](lifecycle-registries.md).
- Permissions for commands live in `requirements.md`, not in `plugin.yml`, under Brigadier.
- For the vanilla permission nodes a server already has:
  `paper/admin/reference/vanilla-command-permissions.md` (175 lines).

## Do not read

`paper/dev/api/command-api/index.mdx`, `.../basics/index.mdx`, `.../arguments/index.mdx`,
`.../misc/index.mdx` — 8–18 line sidebar stubs.
