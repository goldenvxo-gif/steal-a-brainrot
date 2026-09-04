# noob steal a brainrot

Source scripts for the Roblox place **noob steal a brainrot** (`placeId: 80230497814154`),
exported directly from Roblox Studio.

Every `.lua` file here was copied byte-for-byte out of the live place. Models, meshes,
GUIs and other non-script instances are **not** in this repo — this is a code backup and
history, not a full place backup. Keep publishing the place from Studio as usual.

## Naming

Rojo conventions, so this can be synced back into Studio later:

| Suffix | Studio class |
| --- | --- |
| `.server.lua` | `Script` |
| `.client.lua` | `LocalScript` |
| `.lua` | `ModuleScript` |

Folder paths mirror the Studio hierarchy: `src/ServerScriptService/Modules/BaseManager.lua`
is `game.ServerScriptService.Modules.BaseManager`.

## What's in here

### `src/ReplicatedStorage/Modules` — shared config, readable by client and server

| File | What it does |
| --- | --- |
| `BrainrotConfig.lua` | The rarity table and the 87-entry Brainrot catalog. Every entry has a permanent `id`, so you can swap names and models later without breaking anyone's save. |
| `GameConfig.lua` | Tunable numbers in one place: lock durations, conveyor interval, offline income, cooldowns, tool ranges. |
| `ShopConfig.lua` | Base upgrades, tools and utility boosts with their costs and levels. |
| `RarityUtils.lua` | Weighted rarity rolling, including the luck multiplier. |
| `Numbers.lua` | Number formatting (1.2K, 3.4M...). |

### `src/ServerScriptService` — the game logic

`GameManager.server.lua` is the entry point; it initializes every module below and wires
up player join/leave.

| Module | What it does |
| --- | --- |
| `BaseManager.lua` | Assigns a base per player, owner nameplates, spawn placement, cleanup on leave. |
| `LockManager.lua` | Base shields: auto-lock on join, manual lock/unlock prompt, per-base collision groups. |
| `ConveyorManager.lua` | Spawns Brainrots onto the belt, moves them, despawns at the end. Handles Rainbow variants. |
| `CarryManager.lua` | Pickup, carrying, depositing into a slot, force-drop on death or bat hit. |
| `StealManager.lua` | Stealing from unlocked enemy bases, per-victim cooldowns, raid alerts. |
| `CombatManager.lua` | Server-authoritative bat swings: range check, knockback, ragdoll, drop. |
| `IncomeManager.lua` | Passive coin income ticker, with rebirth multiplier. |
| `DataManager.lua` | DataStore save/load, offline earnings, slot restore. Degrades safely in an unpublished place. |
| `CatalogManager.lua` | Tracks which Brainrots (and which Rainbows) each player has discovered. |
| `PreviewBuilder.lua` | Builds lightweight render-only model copies for the catalog's 3D previews. |
| `RebirthManager.lua`, `SecurityManager.lua`, `ShopManager.lua` | Empty placeholders — not written yet. |

### `src/StarterPlayer` — client UI

| File | What it does |
| --- | --- |
| `StarterPlayerScripts/CatalogClient.client.lua` | The Brainrot Index: filterable grid, 3D previews, detail cards, discovery toasts. The biggest client script. |
| `StarterPlayerScripts/CoinHUD.client.lua` | Coin counter, bottom-centre. |
| `StarterPlayerScripts/StealNotificationClient.client.lua` | Raid and "your Brainrot was stolen" toasts. |
| `StarterPlayerScripts/OfflineEarningsClient.client.lua` | "Welcome back" offline earnings popup. |
| `StarterPlayerScripts/CombatFeedback.client.lua` | Red flash when hit. |
| `StarterCharacterScripts/CarryClient.client.lua` | "Carrying: ..." label. |
| `StarterPlayerScripts/GameClient.client.lua`, `StarterCharacterScripts/AnimationController.client.lua` | Placeholders. |
| `StarterPlayerScripts/ClientModules/*` | Empty placeholders. |

### `src/StarterPack/Bat`

`BatClient.client.lua` — swing animation, cooldown and handle flash. All hit detection is
server-side in `CombatManager`.

### `src/Workspace` and `src/ServerStorage`

Scripts that came with the base kit: the three leaderboards, the two shop NPC prompts,
`GuaranteedRarities`, and a per-model animation script.

## Known issues in the exported code

These are real problems in the current place, kept here so they don't get lost:

- `Workspace/Game/GuaranteedRarities` and both shop scripts `require` modules that don't
  exist in this place (`ServerStorage.Modules.Things`, `ServerStorage.Configuration.Modules`,
  `ReplicatedStorage.Configuration.Modules`). `Insert` and `SetProperties` in
  `ServerStorage/Modules` are empty stubs, so those three scripts error on start.
- `RebirthManager`, `SecurityManager`, `ShopManager` and all four `ClientModules` are empty
  `return {}` stubs. `GameManager` doesn't require the first three, so that's harmless today.
- **Four scripts were deliberately left out of this repo.** See `SECURITY-NOTES.md` —
  read that one first.

## Syncing back into Studio

`default.project.json` is set up for [Rojo](https://rojo.space). With Rojo installed:

```
rojo serve
```

then connect from the Rojo plugin in Studio. Note the project file only maps the folders
holding your own code; the `Workspace` kit scripts are exported for reference but not
mapped, because they live inside models that Rojo would have to own.
