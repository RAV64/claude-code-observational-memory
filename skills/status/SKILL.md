---
description: Show the current state of observational memory for this project - ledger size, watermarks, memory files. Use when the user asks about memory state or cc-om health.
allowed-tools: Bash, Read
---

Run `om status` from the project root (`"${CLAUDE_PLUGIN_ROOT}/bin/om"` if `om` is not on PATH) and present the output to the user:

- List the durable memory files with their one-line purpose from `MEMORY.md`.
- Report each listed session's observation ledger size and watermark.
- If the ledger is near the consolidation threshold, say so.

If the user asks about the content of a specific topic, read that file from the memory directory and summarize it.
