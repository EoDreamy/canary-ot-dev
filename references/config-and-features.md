# config.lua defaults and features (Canary)

Canary is configured via `config.lua` (a copy of `config.lua.dist` at the
repo root), loaded through `src/config/`. Always diff the actual
`config.lua.dist` in the target repo rather than trusting this snapshot —
defaults change as the project evolves.

## Datapack selection

```lua
useAnyDatapackFolder = false
dataPackDirectory = "data-otservbr-global"  -- or "data-canary"
coreDirectory = "data"
```

`useAnyDatapackFolder` lets the server load an arbitrary datapack folder
name when `true`. `worldType` (`"pvp"`, `"no-pvp"`, `"pvp-enforced"`)
defaults to `"pvp"`.

## Notable feature toggles (verify current values/keys before relying on them)

- `generateLuaApiDocs` / `luaApiDocsOutputDirectory` — drives the Lua API
  doc generator (`src/lua/docgen/`).
- Weapon Proficiency system: `weaponProficiencyMaxLevels`,
  `weaponProficiencyMaxPerksPerLevel`, `weaponProficiencyGainMultiplier`
  (1.0 = normal progression, values scale XP gain; see
  `docs/systems/weapon-proficiency.md` if present).
- Wheel of Destiny: `transcendenceAvatarDuration` and related keys.
- Forge system: `forgeInfluencedIntervalTime` / `forgeInfluencedIntervalType`
  control how often influenced monsters spawn.
- Animus Mastery / SoulPit and Augments system: search `config.lua.dist`
  for the relevant section headers — both are configurable subsystems with
  their own key blocks.
- `maxInboxItems` (0 = uint32 max), `maxExivaWhitelist`.
- Player base critical chance/damage: `playerBaseCriticalChance`,
  `playerBaseCriticalDamage`.
- Legacy protocol ports: `legacy1100GameProtocolPort`,
  `legacy860GameProtocolPort` (tied to the multiprotocol system — set to 0
  to auto-select the next available port).
- `buyBlessCommandFee`, `quickLootMaxCorpses`, `depotChest`,
  `monkQuestTotalShrines` / `wheelMonkQuestBonus`.
- `loginProtectionTime` (milliseconds).

## Practical rule

When asked "what's the default for X" or to change a gameplay constant,
**grep the live `config.lua.dist`** (or the server's actual `config.lua` if
one exists) rather than quoting a value from memory — key names, types, and
defaults change across releases.
