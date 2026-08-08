# Adventure essentials — condensed reference

> **Sources and blobs** (see `../bootstrap.md` §4; mismatch means this digest may be behind):
> | Upstream file | Blob |
> |---|---|
> | `adventure/text.md` | `045c7aea2a80978aa312588d6247af3412a2fa07` |
> | `adventure/audiences.md` | `584d5fba7784639e8018c4f77d00a7f887b58a4e` |
> | `adventure/minimessage/api.mdx` | `40d60ce82f77c2d4a977963c8e821eef0e3c216e` |
> | `adventure/minimessage/dynamic-replacements.md` | `d65803ba44fd2cb89ba9e7e37da8366c9636f3c4` |
>
> Replaces ~530 lines. Tag syntax lives in `minimessage-format.md`.

## Components

`Component` is Minecraft's rich-text unit. Immutable — every method returns a new instance.

```java
Component msg = Component.text("You're a ")
  .color(TextColor.color(0x443344))
  .append(Component.text("Bunny", NamedTextColor.LIGHT_PURPLE))
  .append(Component.text("! Press "))
  .append(Component.keybind("key.jump")
      .color(NamedTextColor.LIGHT_PURPLE)
      .decoration(TextDecoration.BOLD, true))
  .append(Component.text(" to jump!"));
```

A mutable builder form exists via `Component.text()` … `.build()` — same result, better for
loops and conditionals.

- **Colour:** `TextColor.color(0xRRGGBB)` for any RGB, `NamedTextColor.*` for the vanilla 16.
- **Decorations:** `BOLD`, `ITALIC`, `UNDERLINED`, `STRIKETHROUGH`, `OBFUSCATED`.
- **Component kinds:** `text`, `translatable`, `keybind`, `score`, `selector`, `nbt`.
- **Events:** `HoverEvent` (text/item/entity) and `ClickEvent` (open URL, open file, run command,
  suggest command, change page, copy to clipboard, open dialog, custom payload).

> **Italic gotcha (not in the docs, bites everyone):** item display names and lore render
> italic by default. Add `.decoration(TextDecoration.ITALIC, false)` explicitly.

## Audiences

`Audience` is the universal target for anything you can send: a player, the console, a command
sender, or a group of them.

```java
audience.sendMessage(component);
audience.sendActionBar(component);
audience.showTitle(title);
audience.playSound(sound);
```

- On Paper, Bukkit types **are** Audiences already — `Player`, `CommandSender`, `Server`, `World`
  all implement it. No adapter needed.
- Compose with `Audience.audience(a, b, …)` or implement `ForwardingAudience`.
- `Audience.empty()` is the no-op.
- **Degrades gracefully:** sending a boss bar to the console does nothing rather than throwing.
  That is the design, and it is why you can write to an `Audience` without type-checking.
- **Pointers** expose data generically:
  ```java
  audience.get(Identity.UUID);                       // Optional<UUID>
  audience.getOrDefault(Identity.DISPLAY_NAME, fallback);
  ```

## MiniMessage

```java
MiniMessage mm = MiniMessage.miniMessage();          // standard: all tags, lenient
player.sendMessage(mm.deserialize("<red>Hello <bold>world"));
```

Round-trips both ways — `deserialize` (string → Component) and `serialize` (Component → string).

**Create one instance centrally and reuse it.** The exception is per-permission configurations
(e.g. admins may use colour tags, ordinary players may not).

### Presets — use these for untrusted input

| Preset | Effect |
|---|---|
| `DEFAULT` | Everything. Same as `MiniMessage.miniMessage()` |
| `NON_INTERACTABLE` | No click events, hover events, or insertion. Post-processes the result to strip interactables |
| `FORMATTED_TEXT` | Text and formatting only — colour, shadow, font, decoration. Strips non-text and interactable components |

```java
MiniMessage safe = MiniMessage.miniMessage(MiniMessage.Preset.NON_INTERACTABLE);
```

Both restricted presets post-process, so they hold even when custom resolvers are added.

### Builder

```java
MiniMessage mm = MiniMessage.builder()
  .tags(TagResolver.builder()
    .resolver(StandardTags.color())
    .resolver(StandardTags.decorations())
    .build())
  .build();
```

Tags not enabled are rendered as **literal text**, not errors.

### Error handling

MiniMessage never throws on user input by default — invalid tags become plain text. `strict(true)`
throws on unclosed tags. `debug(Consumer<String>)` captures why a parse behaved as it did.

## Placeholders and formatters

Pass resolvers as extra args to `deserialize`. **This is the correct way to inject dynamic
values — never string-concatenate into MiniMessage input.**

```java
mm.deserialize("<gray>Hello <name>", Placeholder.component("name", nameComponent));
mm.deserialize("<gray>Hello <name>", Placeholder.unparsed("name", untrustedInput));
mm.deserialize("<gray>Hello <name>", Placeholder.parsed("name", "<red>TEST"));
mm.deserialize("<my-style>Hi</my-style>", Placeholder.styling("my-style",
    ClickEvent.suggestCommand("/say hello"), NamedTextColor.RED, TextDecoration.BOLD));
```

| Resolver | Use for |
|---|---|
| `Placeholder.component(k, Component)` | Insert a prebuilt component |
| `Placeholder.unparsed(k, String)` | **Untrusted text.** Tags inside are not parsed — this is the sanitiser |
| `Placeholder.parsed(k, String)` | Trusted text whose tags should apply (and can leak into following text) |
| `Placeholder.styling(k, …)` | Define your own styling tag from colours/decorations/events |

`Placeholder.component` and `Placeholder.unparsed` are **self-closing** by default.

Formatters:

```java
Formatter.number("no", 250.25d);   // <no>, <no:'#.00'>, <no:'de-DE':'#.00'>
Formatter.date("date", LocalDateTime.now(ZoneId.systemDefault()));  // <date:'yyyy-MM-dd HH:mm:ss'>
Formatter.choice("choice", 5);     // <choice:'0#no developer|1#one developer|1<many developers'>
```

Number patterns can embed style:
`<no:'en-EN':'<green>#.00;<red>-#.00'>` — green when positive, red when negative.

### Custom tags

```java
TagResolver.resolver("click-by-version", (args, ctx) -> {
  final String version = args.popOr("version expected").value();
  return Tag.styling(ClickEvent.openUrl("https://jd.papermc.io/adventure/" + version + "/"));
});
```

Three `Tag` kinds: **PreProcess** (substitutes raw MiniMessage before parsing), **Inserting**
(yields a component or style — what you almost always want, via `Tag.styling(...)`), and
**Modifying** (visits and transforms the whole tree; how `<rainbow>`/`<gradient>` work — return a
fresh instance per resolve, since these hold state). Tag names must match `[a-z0-9_-]+` and may
start with `!`, `?`, or `#`. Details in `adventure/minimessage/api.mdx`.

## Serializers

| Need | Use |
|---|---|
| Plain text (logs, comparison) | `PlainTextComponentSerializer` |
| Vanilla JSON | `JSONComponentSerializer` / `GsonComponentSerializer` |
| Coloured console | `ANSIComponentSerializer` |
| `§` interop with legacy plugins | `LegacyComponentSerializer` — **deprecated**, boundary use only |

Convert legacy input **once at the edge**, then work in Components.
