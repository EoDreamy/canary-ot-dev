# Building and Docker (Canary)

Canary uses **CMake + vcpkg + Ninja/MSVC**, with CMake presets as the
intended entry point (`CMakePresets.json` + `CMakeLists.txt` at the repo
root). Don't invent build commands or ad-hoc build directories — inspect
the repo's own preset file first.

## Windows build workflow

Run from a **Visual Studio Developer Command Prompt/PowerShell** (needed so
`cl.exe` and Ninja are already on `PATH`; if not already in one, run
`VsDevCmd.bat` first):

```bat
cmake --preset windows-release
cmake --build --preset windows-release --target canary
```

Prefer the release preset over the generated `.sln` for normal local
validation — it uses the intended cache/unity build settings. Only fall
back to `vcproj/*.sln` if explicitly needed, and don't switch to it just
because a CMake configure failed — fix the preset environment/cache first.

Signs of an existing CMake/Ninja cache to be careful around:
`CMakeCache.txt`, `CMakeFiles/`, `build.ninja`, `.ninja_deps`, `.ninja_log`,
`cmake_install.cmake`, `compile_commands.json`, `vcpkg-manifest-install.log`,
`VSInheritEnvironments.txt`. If CMake reports a stale/incompatible cache,
delete only the affected preset directory under `build/`, then rerun —
don't create new ad-hoc build trees.

## Adding/removing/renaming C++ files

Update every maintained build entry point in the same change:

- The relevant `CMakeLists.txt`.
- The tracked Visual Studio project, `vcproj/canary.vcxproj` — missing
  `.cpp` entries there show up as unresolved-external linker errors when
  building `vcproj/canary.sln`.
- The relevant test `CMakeLists.txt` under `tests/` if the change affects
  test sources.

## Precompiled headers (`src/pch.hpp` / `src/pch.cpp`)

- Don't add unguarded standard/library `#include`s that `pch.hpp` already
  provides.
- If a translation unit needs a PCH-provided include in a non-PCH build,
  guard it:

```cpp
#ifndef USE_PRECOMPILED_HEADERS
    #include <memory>
#endif
```

- Adding a new broadly-used header to PCH-enabled builds: add it to
  `pch.hpp`, and keep the local fallback include guarded the same way.

## vcpkg dependencies

Core deps include `mbedtls`. OpenTelemetry metrics
(`opentelemetry-cpp` with `otlp-http`/`prometheus` features) is gated
behind an optional `metrics` **feature** — don't assume it's compiled in
by default. A `tests` feature pulls in `gtest`. Check `vcpkg.json` and its
`builtin-baseline` before adding or upgrading a dependency.

## Docker quickstart

`docker/` ships `Dockerfile.arm`, `Dockerfile.x86`, `Dockerfile.dev`,
`docker-compose.yml`, `config.sh`, `.env.dist`, and `DOCKER.md`. The
quickstart is meant for non-expert local use (LAN demos, local dev/test),
**not** a hardened production deployment with default settings:

- Services: `db`, `server`, `myaac` (website/admin AAC only — its
  `login.php` is intentionally excluded), `login-server` (dedicated client
  login webservice). Named volumes `db-volume`/`server-data` on network
  `canary-net`.
- Client login URL: `http://localhost:8088/login`.
- Web/admin URL: `http://localhost:8080`.
- Game port: `7172`.
- `login-server` is the only thing clients should point at for login — do
  not reintroduce MyAAC's `login.php` as a login endpoint.
- Public env vars use a `CANARY_*` prefix; avoid inventing new public
  `MYSQL_*`/`OT_*`/raw-Lua-config-name env vars.
- The quickstart uses the **published runtime image** — it must not
  require compiling canary locally.
- Guarded helper scripts `up.sh` / `up.ps1` start the stack and clean safe
  Docker leftovers without touching DB volumes; prefer them over raw
  `docker compose` invocations when available.

Treat CI/build Docker, local-dev Docker, and the user-facing quickstart as
separate responsibilities — don't let a change to one silently change
another's contract unless that's the explicit intent.

## Tests

`tests/` at the repo root, wired into CMake + CI. Compile/run tests when a
change is critical, complex, or likely to affect compilation; skip it for
doc/script-only changes that clearly don't touch the build.
