# Topic: text, colour, and messages (Adventure)

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

**Start with the digests.** They condense ~1100 lines of upstream and lose almost nothing an
agent needs.

| Read first | Replaces |
|---|---|
| [`digests/minimessage-format.md`](../digests/minimessage-format.md) | `adventure/minimessage/format.mdx` (577) |
| [`digests/adventure-essentials.md`](../digests/adventure-essentials.md) | `adventure/text.md` (96) + `audiences.md` (44) + `minimessage/api.mdx` (239) + `minimessage/dynamic-replacements.md` (150) |

Go upstream when the digest does not cover it.

## Upstream — Paper's own view

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/component-api/intro.md` | 212 | — | Paper-flavoured component intro; how components reach players on Paper |
| `paper/dev/api/component-api/audiences.md` | 53 | — | Getting an `Audience` from Bukkit types |
| `paper/dev/api/component-api/i18n.md` | 57 | — | Paper's take on translating plugin messages |
| `paper/dev/api/component-api/signed-messages.md` | 114 | 1.19.4+ | `SignedMessage` handling |

## Upstream — Adventure proper

| Path | Lines | Read when |
|---|---|---|
| `adventure/minimessage/format.mdx` | 577 | Digest is insufficient — exact tag edge cases, per-tag examples |
| `adventure/minimessage/api.mdx` | 239 | Deeper `TagResolver` work, `ArgumentQueue` handling, `Modifying` tag internals, parser directives |
| `adventure/minimessage/dynamic-replacements.md` | 150 | Placeholder/`Formatter` edge cases beyond the digest |
| `adventure/minimessage/translator.md` | 65 | MiniMessage-backed translation registry |
| `adventure/text.md` | 96 | Raw `Component` building without MiniMessage |
| `adventure/audiences.md` | 44 | `Audience`, `ForwardingAudience`, pointers |
| `adventure/localization.md` | 124 | `TranslatableComponent`, `GlobalTranslator`, resource bundles |
| `adventure/titles.md` | 37 | Title / subtitle / actionbar, timings |
| `adventure/bossbar.md` | 61 | Boss bars |
| `adventure/sound.md` | 83 | Playing sound, sound stops, emitters |
| `adventure/book.md` | 34 | Written books |
| `adventure/tablist.md` | 37 | Player list header/footer |
| `adventure/resource-pack.md` | 70 | Prompting a resource pack |

## Serializers

| Path | Lines | Read when |
|---|---|---|
| `adventure/serializer/index.md` | 58 | Choosing a serializer |
| `adventure/serializer/plain.mdx` | 31 | Component → plain text (logging, comparisons) |
| `adventure/serializer/json.md` | 27 | Component → vanilla JSON |
| `adventure/serializer/gson.mdx` | 51 | Gson-backed JSON serializer |
| `adventure/serializer/ansi.mdx` | 82 | Coloured console output |
| `adventure/serializer/legacy.mdx` | 53 | **Deprecated.** Only for interop with a plugin that still speaks `§` |

## Notes

- **Never `§`/`&`.** If the user has legacy strings, convert once at the boundary with
  `LegacyComponentSerializer`, then work in Components.
- **User-supplied input is a footgun.** Parsing arbitrary player text with a full MiniMessage
  instance lets them inject `<click:run_command:...>`. Use the `NON_INTERACTABLE` or
  `FORMATTED_TEXT` preset — see the essentials digest.
- **Placeholders:** do not string-concatenate into MiniMessage input. Use `Placeholder.unparsed`
  for untrusted values (`dynamic-replacements.md`).
- Testing MiniMessage without a server: <https://webui.advntr.dev>.

## Do not read

- `adventure/platform/{fabric,neoforge,modded,spongeapi,bungeecord,viaversion,implementing}` —
  other platforms. Paper has Adventure natively; you need none of it.
- `adventure/migration/*` — migrating from BungeeCord Chat / text 3.x / Adventure 4.x. Only if
  the user is explicitly doing that migration.
- `adventure/{index,faq,getting-started,community-libraries}.mdx`,
  `adventure/version-history/`, `adventure/minimessage/index.md`,
  `adventure/platform/{index,native}.md` — stubs and overviews.
- `paper/dev/api/component-api/index.mdx` — 8-line stub.
