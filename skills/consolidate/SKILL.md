---
description: Manually run observational-memory consolidation - promote aged observations from the session ledger into durable topic files. Use when the user asks to consolidate memory now.
allowed-tools: Bash, Read
disable-model-invocation: true
---

Run `om consolidate --cwd "$PWD"` from the project root (`"${CLAUDE_PLUGIN_ROOT}/bin/om"` if `om` is not on PATH) with the Bash tool's `run_in_background` option and wait for the task to finish; its output is in the file the task result names. Add `--all` only if the user asked for the whole ledger to be promoted.

Then read `MEMORY.md` and `JOURNEY.md` in the memory directory named by `om: consolidated into <dir>` and report to the user (naming the session from `om: session <id>`):

1. Which topic files were updated or created.
2. The new journey entry.
3. Any new facts that were pinned.

If the output is `om: nothing to consolidate`, tell the user no whole block could be promoted while keeping the retained tail (`OM_RECENT_OBS_LINES` lines) and how much the ledger holds (`om status`). If the output is `om: consolidation already running for ...`, say another consolidation holds the lock and nothing was promoted.

If the command exits non-zero, show the user its error output and say the ledger was kept intact (a failed consolidator run never discards observations).
