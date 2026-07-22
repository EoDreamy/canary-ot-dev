---
name: canary-ot-dev
description: Use when working on Canary (opentibiabr/canary) or a close fork of it — a C++20 + Lua OpenTibia server emulator. Covers the C++ engine architecture, Lua gameplay scripting (actions/talkactions/globalevents/spells/etc.), CMake+vcpkg builds, the Docker quickstart stack, and config.lua defaults/features.
---

# Canary OT server development

[Canary](https://github.com/opentibiabr/canary) is an open-source MMORPG
server emulator for Tibia-based games, written in **C++20** (engine) and
**Lua** (gameplay scripting). It's a fork of
[OTServBR-Global](https://github.com/opentibiabr/otservbr-global).

This skill teaches the project's architecture, conventions, and known sharp
edges so changes can be made safely without re-deriving them from scratch
each time. It does **not** memorize exact file contents or line numbers —
canary changes fast; always confirm specifics in the live repo with
`Grep`/`Glob`/`Read` before relying on them.

## Repository identity check

Before applying this skill's specifics, confirm you're actually in Canary
(or a fork close enough that these hold): look for `AGENTS.md` at the repo
root, `docs/architecture.md`, and an executable named `canary_server.cpp` /
`canary_server.hpp` under `src/`. If those are missing or clearly different,
treat this skill's project-specific details as unreliable and read the
repo's own docs instead.

## Read `AGENTS.md` first

If the repo has an `AGENTS.md` at its root, it is the authoritative,
binding policy for this project — git safety, commit style, build workflow,
the precompiled-header rule, the Lua shared-userdata ownership gate, Docker
quickstart boundaries, and PR comment policy. Read it before making changes,
and prefer it over anything generic in this skill if the two ever disagree
(this skill may be stale; a committed `AGENTS.md` in the repo you're editing
is not).

## Where to look

| Concern | Read this first |
|---|---|
| C++ engine layout (game/creatures/network/map/io/dispatcher) | `references/architecture.md` |
| Writing/editing Lua scripts (actions, talkactions, globalevents, spells, events, revscriptsys) | `references/lua-scripting.md` |
| Building (CMake presets, vcpkg, PCH), Docker quickstart | `references/build-and-docker.md` |
| `config.lua.dist` defaults, datapack folders, feature toggles | `references/config-and-features.md` |

## Shared engine mental model

Canary is event-driven and split cleanly into:

```
Client ↔ ProtocolGame/ProtocolLogin (network) → Dispatcher/Scheduler (game thread)
  → Game singleton → Creatures/Map/Combat/Items → Lua events (data/scripts, data/events)
  → IO layer → MariaDB/MySQL (schema.sql)
```

- **C++ core** owns performance-critical systems: networking, pathfinding,
  combat math, map/tile management, persistence, scheduling.
- **Lua layer** owns gameplay content: NPCs, quests, actions, movements,
  talkactions, spells, custom systems — loaded from `data/scripts` (shared)
  plus a per-flavor datapack folder (`data-canary` or `data-otservbr-global`),
  selected via `dataPackDirectory` in `config.lua`.
- Gameplay is customized in Lua **without recompiling**; only new
  C++-exposed primitives require touching `src/lua/functions/**` and a
  rebuild.

For anything deeper than this summary, open the matching reference file
above rather than guessing — the codebase is large enough that specifics
(exact class names, exact config keys) must be confirmed in-repo.
