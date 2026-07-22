# canary-ot-dev

A [Claude Code Skill](https://code.claude.com/docs/en/skills)
that teaches Claude the architecture and conventions of
[Canary](https://github.com/opentibiabr/canary), a C++20 + Lua OpenTibia
server emulator, so it can make safer, better-informed changes without
re-deriving the codebase's structure from scratch every session.

## What it covers

- **C++ engine architecture** — the `Game` singleton, dispatcher/scheduler,
  creature hierarchy, network protocols, map system, persistence layer.
- **Lua gameplay scripting** — the binding layer under `src/lua/`, the
  "revscriptsys" registration pattern (actions/talkactions/globalevents/
  spells/etc.), and the Lua shared-userdata ownership gate.
- **Build & Docker** — the CMake + vcpkg preset workflow, precompiled-header
  rules, and the Docker quickstart stack.
- **Config & features** — `config.lua.dist` defaults, datapack folder
  selection, and notable gameplay-feature toggles.

This skill intentionally avoids hardcoding exact line numbers or file
contents that will drift as Canary evolves. Instead it teaches Claude
*where to look* and *what to verify* before acting, and to defer to the
target repo's own `AGENTS.md` / docs when present.

## Install

Copy this folder into your Claude Code skills directory, keeping the
`canary-ot-dev/` folder name:

```sh
# Project-scoped (only active in one project)
cp -r canary-ot-dev /path/to/your/canary-project/.claude/skills/canary-ot-dev

# User-scoped (active in every project)
cp -r canary-ot-dev ~/.claude/skills/canary-ot-dev
```

Claude Code will pick it up automatically based on `SKILL.md`'s
`description` whenever you're working in a Canary (or close fork) repo.

## Structure

```
canary-ot-dev/
├── SKILL.md                       # entry point Claude reads first
└── references/
    ├── architecture.md            # C++ engine layout
    ├── lua-scripting.md           # Lua bindings & scripting conventions
    ├── build-and-docker.md        # CMake/vcpkg/Docker workflow
    └── config-and-features.md     # config.lua defaults & feature toggles
```

## Contributing

Canary is actively developed, so this skill will drift out of date. PRs
updating `references/*.md` to match reality are welcome — see
[CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
