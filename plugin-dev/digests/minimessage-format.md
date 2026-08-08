# MiniMessage format — condensed reference

> **Source:** `adventure/minimessage/format.mdx`
> **Source blob:** `a16f068fe32170e37bcf53877fdde7ea7feacabd`
> **Covers:** all 22 standard tag sections documented upstream. Replaces a 577-line read.
> If the blob SHA no longer matches (see `../bootstrap.md` §4), or you need a tag's exact edge-case
> behaviour, read the upstream file.

## Syntax rules

- Everything is a tag: `<tag>`, `<tag:arg>`, `<tag:arg1:arg2>`.
- **Closing tags are optional** outside strict mode. These are identical:
  `<yellow>Hello <blue>World<yellow>!` · `<yellow>Hello <blue>World</blue>!`
- **Self-closing:** `<tag/>` for content-less tags. Required form in strict mode.
- **Case-insensitive** tag names. Keep them lowercase anyway.
- **Quotes:** `'` and `"` are interchangeable. Switch quote style to avoid escaping.
- **Escaping:** `\<` escapes a tag opener in plain text; inside a quoted string escape the quote
  char; `\\` yields a literal backslash. **Unquoted tag arguments cannot contain escapes.**
- **Strict mode** (opt-in): forbids `<reset>`, requires all tags closed in reverse order of
  opening. Default mode silently ignores invalid tags and auto-closes at end of input.
- Tag names may contain `a-z 0-9 _ -` and may start with `!`, `?`, or `#`.

## Colour and style

| Tag | Aliases | Arguments | Notes |
|---|---|---|---|
| `<colorname>` | — | — | `black`, `dark_blue`, `dark_green`, `dark_aqua`, `dark_red`, `dark_purple`, `gold`, `gray`, `dark_gray`, `blue`, `green`, `aqua`, `red`, `light_purple`, `yellow`, `white`. `grey`/`dark_grey` accepted. Hex `#RRGGBB` works directly |
| `<color:X>` | `colour`, `c` | named colour or hex | Verbose form of the above |
| `<shadow:X[:alpha]>` | — | named/hex `#RRGGBB` or `#RRGGBBAA`; alpha float 0–1 | Text shadow colour. Alpha defaults **0.25**; ignored if the hex already carries alpha. `<!shadow>` disables (= `#00000000`) |
| `<decoration>` | see below | `[:false]` to disable | `bold`(`b`), `italic`(`em`,`i`), `underlined`(`u`), `strikethrough`(`st`), `obfuscated`(`obf`). `<!name>` inverts |
| `<reset>` | — | none | Closes all open tags. **Cannot be closed. Forbidden in strict mode** |
| `<font:key>` | — | namespaced key | Defaults to the `minecraft` namespace. E.g. `<font:uniform>`, `<font:myfont:custom_font>` |

## Gradients and animated colour

| Tag | Arguments | Notes |
|---|---|---|
| `<rainbow[:[!]phase]>` | optional phase; leading `!` reverses | `<rainbow:!2>` = reversed, phase 2 |
| `<gradient:[c1]:[c2]:…[:phase]>` | 1..n named/hex colours, optional phase **-1..1** | Colour varies across the text. Phase shifts it — animate by stepping phase |
| `<transition:[c1]:[c2]:…[:phase]>` | same as gradient | Whole text is **one** colour; phase selects which. Not a spatial gradient |
| `<pride[:flag\|phase]>` | flag name or phase | v4.18.0+. Flags: pride, progress, trans, bi, pan, nb, lesbian, ace, agender, demisexual, genderqueer, genderfluid, intersex, aro, baker, philly, queer, gay, bigender, demigender, femboy, intersex inclusive |

## Interaction

| Tag | Arguments | Notes |
|---|---|---|
| `<click:action:value>` | action, value | Actions incl. `run_command`, `suggest_command`, `copy_to_clipboard`, `open_url`, `change_page`. ⚠ Since 1.19.1 the client **refuses** `run_command` for commands needing signed args (`/say`, `/tell`) |
| `<hover:action:value>` | action, value | See the hover table below |
| `<insert:text>` | text | Shift-click inserts the text into the player's chat box |

### `hover` actions

| Action | Value | Meaning |
|---|---|---|
| `show_text` | `text` | A MiniMessage string — nested formatting works |
| `show_item` | `type[:count[(:componentKey:componentValue)…]]` | Item `Key`, optional count, then data-component pairs |
| `show_item` *(legacy)* | `type[:count[:tag]]` | SNBT tag form. **Deprecated**, may be removed — prefer the component form |
| `show_entity` | `type:uuid[:name]` | Entity type `Key`, UUID, optional custom name |

Example: `<hover:show_item:diamond_sword:1:enchantments:'{sharpness:3,knockback:2}'>Sharp!</hover>`

## Content insertion

| Tag | Aliases | Arguments | Notes |
|---|---|---|---|
| `<lang:key[:v1:v2…]>` | `tr`, `translate` | translation key + placeholder values | Renders in the **player's** locale. Values fill the JSON `with` array |
| `<lang_or:key:fallback[:v1…]>` | `tr_or`, `translate_or` | key, fallback text, values | **1.19.4+.** Fallback when the key is missing |
| `<key:keybind>` | — | keybind id | Shows the player's actual bound key, e.g. `<key:key.jump>` |
| `<newline>` | `br` | none | Line break. Works inside `hover:show_text` too |
| `<selector:sel[:separator]>` | `sel` | selector, optional separator | v4.11.0+. E.g. `<selector:@e[limit=5]>` |
| `<score:name:objective>` | — | holder name or selector, objective | v4.13.0+. **Requires server-side rendering** to be visible |
| `<nbt:src:id:path[:sep][:interpret]>` | `data` | `block`\|`entity`\|`storage`, id, NBT path | v4.13.0+. **Requires server-side rendering.** `interpret` parses the result as component JSON |
| `<sprite[:atlas]:sprite>` | — | atlas, sprite | v4.25.0+. E.g. `<sprite:blocks:block/stone>` |
| `<head:name\|uuid\|texture[:outer_layer]>` | — | identifier, optional bool | v4.25.0+. `outer_layer` defaults `true` |

## Gotchas

- **`transition` is not `gradient`.** Gradient spreads colours across characters; transition
  colours everything uniformly and uses phase to pick the colour.
- **`score` and `nbt` need rendering** — a platform-specific server-side step. They will not
  simply appear.
- **Never interpolate untrusted text into a MiniMessage string.** A player who types
  `<click:run_command:/op them>` gets it parsed. Use `Placeholder.unparsed`, or a restricted
  preset — see `adventure-essentials.md`.
- Test without a server: <https://webui.advntr.dev>.
