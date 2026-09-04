# Security notes — read this before touching the Map folder

While exporting, four scripts in `Workspace.Map` turned up that are **not** part of your
game and should not be trusted. They are deliberately excluded from this repo.

## What was found

Two near-identical script pairs, both dressed up as a "texture management system" with
long, friendly comments about texture presets and lighting:

| Script | Line | What it actually does |
| --- | --- | --- |
| `Workspace.Map.CoreTextureSystem` | 265 | `require(...)` an **asset ID hidden in an attribute** named `Version` on its child `TextureConfiguration` (value `109195943726360`) |
| `Workspace.Map.VisualCoreRepresentation` | 262 | `require(...)` an **asset ID hidden in a child value object** named `Pose` under `CoreConfiguration` (value `74665384229936`) |

`require(<number>)` loads and runs a model from the Roblox catalog **at runtime**. The
person who owns that asset can change what it does at any time, and it runs on your server
with full permissions. Hiding the ID in an attribute or a child value object — rather than
writing it in the code — is done specifically so a search of the source doesn't find it.
This is the standard shape of a Roblox backdoor.

Alongside them:

| Script | What it does |
| --- | --- |
| `Workspace.Map.CoreTextureSystem.LocalScript` | Sets `script.Parent = nil` to hide itself from the Explorer, then disables **every** CoreGui element — including the player list, chat, and the report button |

That last one is exported in this repo (`src/Workspace/Map/CoreTextureSystem/LocalScript.client.lua`)
because it's short and worth being able to read. The two loaders and their two config
modules are not, because publishing a working backdoor to GitHub isn't a good idea.

## What to do

1. In Studio, delete these from `Workspace.Map`:
   - `CoreTextureSystem` (and its `TextureConfiguration` child and its `LocalScript`)
   - `VisualCoreRepresentation` (and its `CoreConfiguration` child)
2. Search the whole place for `require(` followed by a number, and for `getfenv`,
   `loadstring`, and `HttpGet`. Legitimate code in this place only ever requires modules
   by path.
3. Check `ServerScriptService`, `ServerStorage` and `ReplicatedStorage` for anything you
   don't recognise, especially scripts with names that sound like infrastructure
   ("CoreValidation", "TextureSystem", "Analytics", "Handler").
4. If the place has ever been published and run with these in it, change any
   account passwords you reuse elsewhere and check the place's collaborators list.

## Where they came from

Free-model kits. The same pattern showed up before in the Slap tool
(`CoreValidation`). When adding models from the toolbox, it's worth expanding every
script inside them before dragging them into the place.
