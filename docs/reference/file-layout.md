# File layout reference

## Plugin source

```
cc-om/
├── .claude-plugin/plugin.json   manifest
├── hooks/hooks.json             lifecycle wiring
├── bin/om                       engine CLI
├── prompts/observer.md          observer prompt (the `claude -p` prompt argument; the chunk is stdin)
├── prompts/consolidator.md      consolidator prompt (the `claude -p` prompt argument; the observations are stdin)
├── skills/{status,consolidate,remember}/SKILL.md
├── .claude/                     companion settings and CLAUDE.md for this repo
└── docs/                        this documentation
```

Details: [hooks](hooks.md), [CLI](cli.md), [install](../how-to/install.md).

## Data root

`OM_DATA_ROOT` ([configuration reference](configuration.md)).

## Session state (`<data root>/sessions/<session_id>/`)

`<session_id>` is the payload's session id with every character outside `A-Za-z0-9._` replaced by `-`, or `unknown-<key>` (the memory directory key) when the payload has none or the encoded id is `.` or `..`. Removed by `om finalize` once no recorded transcript exists (none recorded, or the file is gone), the directory has not changed for more than 30 days (every hook event and `om finalize` touch it), and its ledger is below `OM_FINALIZE_MIN_LINES` lines.

| File | Content |
|---|---|
| `observations.md` | The observation ledger. Blocks of `- ` bullets under `### <UTC timestamp> (lines <a>-<b>, <n> bytes)` headers |
| `watermark.lines` | Count of transcript JSONL lines observers have processed; chunks are line-aligned (whole records only) |
| `memdir` | Path of the memory directory this session feeds, fixed at its first hook event |
| `transcript` | Path of the session transcript from the latest hook payload that carried one |
| `observations.pre-compact.md` | Ledger snapshot taken by the PreCompact hook |
| `events.log` | Timestamped pre-compact, drain, session-end, and deferred-finalize events |
| `observe.log` | Observer failures (failed calls with their stderr, empty output, output without observation lines) |
| `consolidate.log`, `finalize.log` | Output of detached consolidation / finalize runs |
| `.consolidating.md` | Ledger snapshot present while a consolidator run is in flight |
| `.observe.lock` | `flock(2)` lock file (empty, persistent) |
| `needs-finalize` | Created when `om finalize` starts and removed when its whole-ledger consolidation succeeds or the ledger is below `OM_FINALIZE_MIN_LINES`, so a killed `finalize` leaves it; also removed on the next successful consolidation of the session, or by another session's `om finalize` (after consolidating the ledger, or at once if it is below `OM_FINALIZE_MIN_LINES`) |

## Consolidation lock (`<data root>/locks/<key>.lock`)

One `flock(2)` lock file per memory directory, shared by all sessions that consolidate into it. `<key>` is the memory directory path with every character outside `A-Za-z0-9._` replaced by `-`.

## Consolidator work directory

`${XDG_CACHE_HOME:-~/.cache}/cc-om/work.<key>/` (same `<key>` as the lock) — copy of the memory directory in which the consolidator model runs; removed after a successful run, kept with a `-failed` suffix and the failure time as its mtime after a failed one (an empty directory if the work directory was gone), replacing the previous failed copy; for 30 minutes after that, `om observe` starts no detached consolidation of this memory directory (failed copies not modified for 3 hours are removed by the next consolidator run for any memory directory). At the start of a run, every process group with a live process carrying `OM_WORK_MARK=<work directory>` (set on each model call for it) is killed.

## User configuration

`OM_ENV_FILE` ([configuration reference](configuration.md)).

## Durable memory directory

Default location: `${CLAUDE_CONFIG_DIR:-~/.claude}/projects/<project-slug>/memory/`, identical to the native auto-memory directory unless `autoMemoryDirectory` or `CLAUDE_CODE_PROJECT_DIR_NAME` is set or the working directory is a linked git worktree (cc-om keys on the worktree root; native auto memory shares one directory across worktrees). `<project-slug>` is the root of the git repository containing the session working directory (hook `cwd`; `$PWD` for manual subcommands, `--cwd` for `consolidate` and `finalize`; a session's recorded `memdir` takes precedence), or that directory itself outside a repository, with every character outside `A-Za-z0-9` replaced by `-`. Overridable via `OM_MEMORY_DIR`.

| File | Content | Written by |
|---|---|---|
| `MEMORY.md` | Index: one line per topic file, `- [Title](file.md) — summary` | consolidator |
| `pinned-facts.md` | Kept at 20 bullets or fewer by the consolidator; entries marked `(pinned)` are user-placed, exempt from the cap, and may push the file over it | `om remember`, consolidator |
| `JOURNEY.md` | Chronological 1–3-sentence entries per consolidated batch | consolidator |
| `<topic>.md` | Dated bullet lists per topic (`architecture.md`, `decisions.md`, ...) | consolidator, hand edits |

## Memory block

Emitted by `om session-start` as `additionalContext`:

```
<observational-memory>
This block is maintained by the cc-om plugin. It survives compaction.
Freshness: <UTC timestamp>. If an earlier copy of this block appears in the conversation (e.g., preserved by a compaction summary), this one supersedes it; ignore older copies.

## Pinned facts
(pinned-facts.md, whole file)

## Journey (recent)
(JOURNEY.md, last 30 lines)

## Memory map (read the listed files in <memory dir> for detail)
(MEMORY.md, first OM_INDEX_HEAD_LINES lines)

## Recent observations (this session)
(observations.md, last OM_RECENT_OBS_LINES lines)
</observational-memory>
```

Sections whose source file is missing or empty are omitted; if no section has content, nothing is injected.
