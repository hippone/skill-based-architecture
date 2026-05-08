# scripts/

CLI tooling for the skill-based-architecture project. Single Node.js file, zero dependencies.

## `skill-asset`

AAR-time consolidation helper. Surfaces existing sections that may overlap with a new lesson, before it's written into a rule file. Pure read-only — never modifies markdown.

### Three commands

| Command | Purpose |
|---|---|
| `where <keyword>...` | Suggest where a new lesson should be placed (ranked candidate sections) |
| `related <keyword>...` | Search all rule files for sections matching keywords |
| `group` | Scan all `##` headings and detect topic-similar sections across different files (duplicate-topic detector) |

### Quick examples

```bash
# When AAR identifies a new lesson, ask: where does this belong?
./scripts/skill-asset where renderAmis 第二参数 createObject

# When you want to see every place a topic is discussed
./scripts/skill-asset related "z-index"

# Periodic doc-health check: any topics scattered across files?
./scripts/skill-asset group
```

### Global flags

| Flag | Default | Purpose |
|---|---|---|
| `--top N` | where=5, related=10, group=20 | Limit number of results |
| `--json` | off | Emit JSON instead of human-formatted output (for piping or hook integration) |
| `--help`, `-h` | — | Print usage and exit |

### Search scope

The CLI looks in this order:

1. `./skills/`, `./rules/`, `./references/`, `./workflows/` (preferred — the canonical layout)
2. Top-level `*.md` (fallback for projects that don't use the structured layout)

### What the CLI does NOT do

- ❌ Detect stale rules (semantic obsolescence is unsolvable by tooling alone)
- ❌ Auto-write or auto-edit anything — it only **surfaces candidates**, the author decides
- ❌ Require frontmatter on rule files — operates on plain markdown content

### Exit codes

| Code | Meaning |
|---|---|
| 0 | Normal (including "no matches found") |
| 2 | Argument error |

## Integration into your workflow

The intended use is during **AAR / Task Closure Protocol**: when a new lesson has been identified and the author is about to record it, run `skill-asset where <keywords>` to surface candidate destination sections and avoid duplicate sections accumulating across files.

Optional ways to wire it in:

- Add a step to your `update-rules.md` workflow: "Before appending, run `./scripts/skill-asset where <keywords>` and check if the lesson belongs in an existing section."
- Run `./scripts/skill-asset group` periodically (e.g., before a release) to catch duplicate-topic sections that have crept in.
- Hook it into a maintenance script (`check-all.sh`-style) so duplicate detection runs on every CI build.

### Group output: heuristic, not authoritative

The `group` command reports **potential** topic-similar sections based on heading-token overlap. **Some matches are intentional**:

- Each `references/<topic>.md` having its own `## Anti-patterns` section is a valid cross-cutting pattern, not a duplicate to merge
- Same `## When To Use` heading across different `executable-skill-architecture.md` and `multi-skill-routing.md` reflects parallel reference structure, not redundancy

The CLI prints a footer disclaimer reminding the user to treat results as **candidates for review**, not commands to merge. Use it as a signal to *look*, not as a rule to *act*.

## Tests

`./scripts/test.js` provides smoke tests covering all commands, flags, error codes, frontmatter handling, BOM stripping, CJK tokenization, and the group disclaimer.

```bash
node scripts/test.js
```

## Requirements

- **Node.js 16+** (uses stdlib + `Intl.Segmenter` for CJK word segmentation)
- On Node 14 / 15, the CLI still runs but falls back to whitespace-only tokenization, which produces poor `group` results for Chinese-heavy projects.

## License

Same as the parent project.
