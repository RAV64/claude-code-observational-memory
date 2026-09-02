# Configuration reference

## Environment variables (read by `bin/om`)

Numeric variables must be non-negative integers.

| Variable | Default | Meaning |
|---|---|---|
| `OM_OBSERVER_MODEL` | `haiku` | Model for observer runs (`claude -p --model ...`) |
| `OM_CONSOLIDATOR_MODEL` | `sonnet` | Model for consolidation runs |
| `OM_CHUNK_BYTES` | `60000` | New transcript bytes that trigger one observer run; also the chunk size. At least `OM_FLUSH_MIN_BYTES` |
| `OM_FLUSH_MIN_BYTES` | `4000` | Smallest transcript tail `observe --flush` processes (session-end flush and each PreCompact drain pass); a smaller tail is left for the next observer |
| `OM_LEDGER_SOFT_LIMIT` | `400` | Ledger line count that triggers detached mid-session consolidation; must exceed `OM_RECENT_OBS_LINES` |
| `OM_RECENT_OBS_LINES` | `60` | Ledger tail kept hot: injected at session start; mid-session and manual consolidation retain at least this many lines, in whole blocks (`--all` promotes the whole ledger) |
| `OM_FINALIZE_MIN_LINES` | `20` | Smallest ledger consolidated at session end; below it, observations stay in the per-session ledger only. Also the prune threshold for stale session state ([file layout](file-layout.md#session-state-data-rootsessionssession_id)) |
| `OM_INDEX_HEAD_LINES` | `100` | Lines of `MEMORY.md` injected at session start |
| `OM_RETRIEVE_MAX_FILES` | `4` | Max topic-file hints injected per user prompt |
| `OM_MEMORY_DIR` | `${CLAUDE_CONFIG_DIR:-~/.claude}/projects/<slug>/memory` | Durable memory directory ([file layout](file-layout.md#durable-memory-directory)) |
| `OM_LOCK_WAIT` | unset | Seconds (a non-negative integer) to block waiting for the observe or consolidation lock; unset or empty means skip immediately if held. The transcript drain (`pre-compact`, `finalize`) sets `120` for its flush passes; `consolidate` sets `300` for its observe-lock takes; finalize defaults to `600` for its consolidation; `remember` waits `600`; mid-session detached consolidation runs with it empty |
| `OM_NESTED` | unset | Set to `1` by `om` on the nested observer and consolidator `claude -p` sessions; when non-empty, the hook subcommands exit immediately |
| `OM_NO_SPAWN` | unset | Set to `1` by `om finalize`; when non-empty, `om observe` does not spawn mid-session consolidation |
| `OM_ENV_FILE` | `${XDG_CONFIG_HOME:-~/.config}/cc-om/env` | Shell file `om` sources at startup; holds credentials for hook contexts (e.g. `CLAUDE_CODE_OAUTH_TOKEN`). Values it sets for `OM_LOCK_WAIT`, `OM_NESTED`, or `OM_NO_SPAWN` are ignored when the variable is already set (as it is on `om`'s own child processes) |
| `OM_DATA_ROOT` | `$CLAUDE_PLUGIN_DATA` | Root of session state and consolidation locks. Without `CLAUDE_PLUGIN_DATA` (manual invocations), the most recently modified `${CLAUDE_CONFIG_DIR:-~/.claude}/plugins/data/cc-om-*` directory that contains `sessions/` (Claude Code names it after `cc-om@<marketplace>` with characters outside `A-Za-z0-9_-` replaced by `-`), then `~/.cc-om` |
| `CLAUDE_PLUGIN_DATA` | set by Claude Code | Default for `OM_DATA_ROOT` in hook contexts |

## Companion settings (must be set by the user)

How to set them: [install](../how-to/install.md#required-set-the-companion-settings). Rationale: [design decisions](../explanation/design-decisions.md#memory-ownership), [compaction](../explanation/design-decisions.md#postpone-compaction-instead-of-controlling-it).

| Key | Value | Effect |
|---|---|---|
| `autoCompactWindow` | `300000` | Auto-compaction window in tokens (100000–1000000, capped at the model's context window) |
| `autoMemoryEnabled` | `false` | Disables native auto memory, including `MEMORY.md` index loading |

## Related Claude Code settings

| Setting / variable | Effect |
|---|---|
| `CLAUDE_CODE_AUTO_COMPACT_WINDOW` | Per-session override of `autoCompactWindow` |
| `DISABLE_AUTO_COMPACT=1` | Turns auto-compaction off for the session; manual `/compact` still works |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY` | `1` disables auto memory, `0` forces it on; either overrides `autoMemoryEnabled` |
| `CLAUDE_CONFIG_DIR` | Relocates Claude Code's `projects/` and `settings.json`; `om` reads the memory directory, settings, and the plugin data directory from it |
