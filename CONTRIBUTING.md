# Contributing

This skill describes a live, actively-developed codebase (Canary), so it
will drift out of date. The most useful contribution is keeping
`references/*.md` matching reality.

## Reporting a stale fact

Open an issue or PR describing what changed: a moved/renamed file, a
removed config key, a new subsystem, a different default. A short "X used
to be at path A, now it's at path B" is enough.

## Submitting a change

1. Verify the change against the actual `opentibiabr/canary` source (or the
   fork you're using) — don't guess.
2. Keep edits scoped to the relevant `references/*.md` file; avoid
   restructuring unless the existing structure no longer fits.
3. If a reference file grows past ~100 lines, keep its `## Contents`
   section in sync.
4. Keep `SKILL.md` itself short — it's the always-loaded entry point, and
   detail belongs in `references/`.

## Adding a new topic

If a whole area isn't covered yet (e.g. a major subsystem with no
reference file), open an issue first so the scope can be agreed on before
writing — this keeps the skill focused rather than sprawling.
