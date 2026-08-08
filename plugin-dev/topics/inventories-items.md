# Topic: inventories, items, and recipes

Paths are relative to `.claude/extern/papermc-docs/src/content/docs/`.

| Path | Lines | Min ver | Read when |
|---|---|---|---|
| `paper/dev/api/inventories/custom-inventory-holder.md` | 158 | — | Building a GUI. **The correct way to identify your own inventories** |
| `paper/dev/api/inventories/menu-type-api.md` | 157 | 1.21.8 | Opening a specific vanilla menu type (anvil, furnace, …) with custom contents |
| `paper/dev/api/data-component-api.md` | 225 | — | Modifying items: names, lore, enchantments, custom model data, food, tools |
| `paper/dev/api/recipes.md` | 91 | — | Registering crafting/smelting recipes, discovering them for players |

## Notes

- **Identify inventories by `InventoryHolder`, not by title.** Titles are Components now and
  comparing them is fragile and breaks under translation. `custom-inventory-holder.md` exists
  precisely because the title-matching approach is wrong.
- **Item data components replaced `ItemMeta` NBT.** `data-component-api.md` is the current API;
  a lot of pre-1.20.5 sample code found elsewhere is obsolete. There is a conversion tool at
  `misc/tools/item-command-converter.mdx` (21 lines) for old NBT-format item commands.
- **Recipes register on enable**, and recipe keys are `NamespacedKey`s owned by your plugin —
  re-registering the same key on reload throws.
- Giving items a hover preview in chat: `show_item` in the MiniMessage digest.

## Do not read

`paper/dev/api/inventories/index.mdx` — 8-line stub.
