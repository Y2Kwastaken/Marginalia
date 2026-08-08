# Topic: particles and dialogs

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

Both files are large and densely factual — **read the named section, not the file.**

## Particles — `paper/dev/api/particles.mdx` (908 lines, min ver 26.2)

Read `## count argument behavior` (line ~41) **every time**. It is short, badged Important, and
governs how every other particle behaves:
- `count = 0` → one particle, offsets are reused as a **direction/velocity** vector scaled by `extra`.
- `count > 0` → `count` particles scattered by a Gaussian over the offsets; `extra` is speed.

Getting this backwards is the single most common particle bug.

| Section | Line | For |
|---|---|---|
| `## Directional particles` | 51 | Particles with initial velocity; `### Random direction` (83), `### Specified direction` (108), full list (163) |
| `## Colored particles` | 218 | `### Dust particles` (251), `#### Dust transition` (293), `### Note` (326), `### Trail` (379) |
| `## Converging particles` | 411 | Enchant-table style inward motion; list at 447 |
| `## Material particles` | 457 | `### BlockData` (458), `### ItemStack` (496) |
| `## Sculk particles` | 529 | `### Sculk charge` (530), `### Shriek` (559), `### Vibration` (584) |
| `## Rising particles` | 615 | Upward drift; list at 655 |
| `## Scalable particles` | 668 | `### Sweep attack` (676), `### Explosion` (706) |
| `## Geyser particles` | 737 | `### Base and poof` (740), `### Plume` (793), `### Emitter` (829) |
| `## Miscellaneous behaviors` | 859 | Angry villager, cloud, damage indicator, dust pillar/plume, firefly, powered, splash |

**Use `ParticleBuilder`, not `spawnParticle()`.** The docs prefer it: reusable, clearer, and it
gives you `receivers(radius, sphere)` for controlling who sees the effect. Line numbers drift on
upstream updates — grep the heading text if they do not match.

## Dialogs — `paper/dev/api/dialogs.mdx` (435 lines, min ver 1.21.8)

| Section | Line | For |
|---|---|---|
| `## Showing dialogs` | 29 | Displaying one to a player |
| `## Built-in dialogs` | 36 | Server links and other ready-made dialogs |
| `## Creating dialogs dynamically` | 48 | `### Dialog base` (69), `### Dialog type` (115) |
| `## Registering dialogs in the registry` | 129 | Persistent, referenceable dialogs |
| `## Closing dialogs` | 156 | Programmatic close |
| `## Example: A blocking confirmation dialog` | 167 | Full worked example, incl. gating join (209) |
| `## Example: Retrieving and parsing user input` | 319 | Input fields; `### Reading the input` (359), `### Using callbacks` (406) |

Dialogs are client UI introduced in 1.21.7. Confirm the user's target version before proposing
them — there is no graceful degradation on older clients.
