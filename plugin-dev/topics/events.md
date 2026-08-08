# Topic: events

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/event-api/event-listeners.md` | 136 | — | Any listener work: `@EventHandler`, `Listener`, priorities, `ignoreCancelled`, registration |
| `paper/dev/api/event-api/custom-events.md` | 152 | — | Defining and firing your own event |
| `paper/dev/api/event-api/handler-lists.md` | 62 | — | `getHandlerList()` and why a custom event needs it; unregistering |
| `paper/dev/api/event-api/chat-event.md` | 143 | — | `AsyncChatEvent` — chat formatting, renderers, cancelling chat |
| `paper/dev/api/component-api/signed-messages.md` | 114 | 1.19.4+ | Handling `SignedMessage`, chat signature and deletion behaviour |

## Notes

- **Hot events.** `PlayerMoveEvent` and `BlockPhysicsEvent` fire enormously often. Cheapest
  guard first (e.g. compare block coords before doing anything), then work.
- **Chat is async.** `AsyncChatEvent` runs off the main thread — you may not touch most of the
  Bukkit API from it. Schedule back to the main thread; see [scheduling-folia.md](scheduling-folia.md).
- **Cancellable ≠ undoable.** Cancelling negates the effect; it does not roll back work you
  already did in the handler.
- Registration happens in `onEnable` via
  `Bukkit.getPluginManager().registerEvents(listener, plugin)`.
- Adding an event to Paper itself (not your plugin) is `paper/contributing/events.md` — that is
  prong 2 territory, not plugin development.

## Do not read

`paper/dev/api/event-api/index.mdx` — 8-line stub.
