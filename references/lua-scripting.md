# Lua scripting in Canary

## Contents

- Binding layer (`src/lua/`)
- Lua shared-userdata rule (critical)
- Gameplay script categories (`data/scripts/`)
- `data/events/` (core event hooks)
- Core Lua bootstrap files
- Datapack folders
- Lua tooling

## Binding layer (`src/lua/`)

```
src/lua/
├── callbacks/    # event_callback_manager — custom callback hooks
├── creature/      # creature/player/monster/npc bindings
├── docgen/        # Lua API doc generator (see below)
├── functions/
│   ├── core/       # Game, Config, DB, Position, etc.
│   ├── creatures/  # Player/Monster/NPC/Creature methods
│   ├── events/     # event-object bindings
│   ├── items/
│   └── map/
├── global/         # global functions/constants exposed to every script
├── modules/
└── scripts/        # C++-side script loader/interface
```

`docgen/` (`lua_api_doc_generator.*`) produces `docs/lua-api/lua_api.md` /
`.json` / `.d.lua` when `generateLuaApiDocs = true` in `config.lua` — this
is a real, generated Lua API reference. Check it (or regenerate it) before
guessing a binding's signature; if it's stale or missing, read the binding
source directly under `src/lua/functions/**`.

## Lua shared-userdata rule (critical, C++ side)

If you touch any C++ binding that stores a `std::shared_ptr<T>` inside Lua
userdata, treat it as high-risk ownership code:

- New shared userdata types must define `LuaUserdataTraits<T>::name` and
  use `Lua::registerSharedClass<T>`, `Lua::pushSharedUserdata<T>`, or
  `Lua::pushBorrowedSharedUserdata<T>`.
- Never pair `Lua::pushUserdata<T>(..., std::shared_ptr<T>)` with a manual
  `Lua::setMetatable` call.
- Never use `Lua::setWeakMetatable` on userdata holding a
  `std::shared_ptr<T>` — it strips `__gc` and can leak the C++ object.
- Never wrap a borrowed object as `std::shared_ptr<T>(&object)` with no
  deleter — use `Lua::pushBorrowedSharedUserdata<T>` instead.

Before/after such a change, run from the repo root:

```sh
rg -n "pushUserdata<.*std::shared_ptr|setWeakMetatable|std::shared_ptr<[^>]+>\\(&" src/lua
rg -n "registerSharedClass\\(L," src/lua
```

If present, read `docs/systems/lua-shared-userdata.md` for the full
rationale before making binding changes.

## Gameplay script categories (`data/scripts/`, shared across datapacks)

```
data/scripts/
├── actions/          # onUse for items (doors, keys, rune-items, etc.)
├── creaturescripts/  # onLogin/onLogout/onDeath/onKill/... hooks
├── eventcallbacks/    # global callback overrides (see events/ below)
├── globalevents/       # server-save, periodic think, startup/shutdown
├── lib/                # shared Lua helper libraries
├── movements/          # onStepIn/onEquip/etc.
├── runes/
├── spells/
├── systems/            # larger custom gameplay systems (bestiary charms,
│                        # concoctions, custom monster loot, item tiers,
│                        # reward chest, discord webhook, ...)
├── talkactions/         # chat commands (!bless, /ban, etc.)
└── weapons/
```

Registration follows a declarative "revscriptsys" pattern — construct an
object, define its callback(s), call `:register()`. Example
(`data/scripts/actions/doors/key_door.lua`, trimmed):

```lua
local keyDoor = Action()

function keyDoor.onUse(player, item, fromPosition, target, toPosition, isHotkey)
    -- ... game logic ...
    return true
end

keyDoor:id(1234)
keyDoor:register()
```

The same shape applies to `TalkAction()`, `GlobalEvent()`, `CreatureEvent()`,
`MoveEvent()`, `Spell()`, etc. — construct, define the relevant `on*`
callback(s), configure matchers (`:id()`, `:words()`, `:event()`,
`:actionid()`, ...), then `:register()`.

## `data/events/` (core event hooks)

`data/events/events.xml` + `data/events/scripts/` wire up lower-level
engine hooks (distinct from the gameplay script categories above — closer
to core server behavior overrides). Check `events.xml` to confirm which
hooks are actually enabled before assuming a script there fires.

## Core Lua bootstrap files (`data/`)

- `data/core.lua` — loads core libraries/constants at startup.
- `data/global.lua` — global helper functions available to all scripts.
- `data/stages.lua` — experience stage table.
- `data/update.lua` — datapack update/migration hooks.

## Datapack folders

Shared `data/` content is supplemented by exactly one selected datapack
folder, chosen via `dataPackDirectory` in `config.lua`:

- `data-canary` — lightweight/custom datapack.
- `data-otservbr-global` — full OTServBR-Global world/monsters/NPCs/raids,
  used by the default Docker quickstart.

`useAnyDatapackFolder = true` allows loading an arbitrary folder name
instead of just these two. Content dropped in the wrong datapack folder
silently won't load — confirm which folder the target server actually
loads before adding monsters/NPCs/raids/quests.

## Lua tooling

`.luarc.json` at the repo root configures the Lua language server
(custom globals/diagnostics). Check it before assuming a generic Lua lint
rule applies as-is.
