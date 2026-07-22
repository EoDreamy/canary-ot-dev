# Canary C++ architecture

## Contents

- Repository layout (`src/`)
- Game singleton
- Scheduler / Dispatcher
- Creature hierarchy
- Network layer
- Map system
- Persistence
- Event system
- Build system
- A note on drift

```
src/
├── account/      # account & session data
├── config/       # config.lua loader / ConfigManager
├── creatures/     # Creature -> Player / Monster / NPC hierarchy
├── database/      # MariaDB/MySQL driver wrapper
├── enums/
├── game/          # Game singleton, scheduling/dispatcher, combat
├── io/            # load/save: players, houses, guilds, market, storage
├── items/
├── kv/            # key-value store (storage/cache-like data)
├── lib/           # third-party/vendored helpers
├── lua/           # Lua binding layer (see lua-scripting.md)
├── map/           # tiles, spawns, pathfinding, houses
├── protobuf/       # protobuf message defs (metrics / external protocols)
├── security/       # RSA, crypto backends
├── server/         # network layer: ProtocolGame / ProtocolLogin / ProtocolStatus
└── utils/
```

`main.cpp` boots `canary_server.cpp/hpp`, the top-level server object.

## Game singleton

`Game` (under `src/game/`) is the central coordinator: world state, player/
creature tracking, event dispatch, combat orchestration, item and map
interaction. Most gameplay code either calls into `Game` or is invoked from
it via a dispatcher task.

## Scheduler / Dispatcher

Asynchronous execution keeps the world thread-safe:

- **Scheduler**: delayed events, timers, creature thinking, regeneration.
- **Dispatcher**: queues game tasks so all game-state mutation happens on
  the game thread, avoiding races.

Flow: `Incoming event → Dispatcher queue → Game thread → World update`.

Canary uses a weighted-deficit round-robin dispatcher
(`src/game/scheduling/dispatcher_wdrr.*`, `dispatcher_budget.*`,
`dispatcher_policy.hpp`, `dispatcher_telemetry.hpp`) plus a dedicated
`monster_compute_service` for parallel monster AI scheduling — check
`docs/systems/performance-lifetime/` if present for the design rationale
before changing scheduling behavior.

## Creature hierarchy

```
Creature
├── Player
├── Monster
└── NPC
```

Shared: health, position, conditions, combat, movement. Specialized
behavior lives in the subclass. Monster AI is split into
`monster_combat_intention`, `monster_pathfinding`, `monster_relevance`,
`monster_targeting` under `src/creatures/monsters/`.

Player-specific state (achievements, badges, cyclopedia, forge history,
storage, titles, VIP, weapon proficiency, Wheel of Destiny) lives in small
component classes grouped under `src/creatures/players/components/**`.

## Network layer (`src/server/network/`)

- **ProtocolLogin**: authentication, character list, session init.
- **ProtocolGame**: movement, combat, chat, inventory updates, world sync.
- **ProtocolStatus**: server status queries.

Canary supports multiple client protocol versions concurrently through a
multiprotocol runtime-profile system (`protocol_profile.*`,
`protocol_session_hint.*`, `transport_codec.*`, `protocol_port_utils.hpp` —
see `docs/systems/multiprotocol.md` if present), including legacy protocol
ports (`legacy1100GameProtocolPort`, `legacy860GameProtocolPort` in
`config.lua.dist`).

## Map system (`src/map/`)

Tile management, visibility checks, pathfinding, spawn management, house
ownership, plus `map_download.*` and `navigation_snapshot.*` for
downloadable/streamed map handling.

## Persistence (`src/io/`, `schema.sql`)

Character/account/guild/house/market load-save logic lives in `src/io/`,
isolated from gameplay logic. Uses MariaDB/MySQL; `schema.sql` at the repo
root is the source of truth for tables (`accounts`, `players`, `guilds`,
`houses`, `items`, `market`, `player_storage`, `player_skills`, etc.).
Player storage access is abstracted behind
`player_storage_repository(.hpp/_db.cpp/.hpp)`.

## Event system

Heavily event-driven: `onLogin`, `onLogout`, `onThink`, `onMove`, `onUse`,
`onDeath`, `onKill`, etc. Events are implemented in Lua wherever possible so
gameplay changes don't require a C++ rebuild — see `lua-scripting.md` for
the registration mechanics.

## Build system

CMake + vcpkg + GitHub Actions, targeting Linux/Windows/macOS. See
`build-and-docker.md` for the concrete command sequence and known pitfalls.

## A note on drift

This document describes the architecture as observed at one point in time.
Canary is actively developed — subsystem names, folder layout, and even
whole systems (scheduling strategy, protocol support, map streaming) can
change. Treat every path and class name here as a **starting hypothesis**
to verify with `Glob`/`Grep`, not a guaranteed fact.
