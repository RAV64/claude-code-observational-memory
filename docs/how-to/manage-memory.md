# How to manage cc-om memory

## Pin a fact that must never be forgotten

In a session:

```
/cc-om:remember never call refreshSession() from the websocket handler - Redis reconnect deadlock
```

Or from the session's Bash tool:

```bash
om remember "never call refreshSession() from the websocket handler - Redis reconnect deadlock"
```

Keep `pinned-facts.md` within its cap ([file layout](../reference/file-layout.md#durable-memory-directory)); remove a pinned entry by editing the file.

## Inspect what memory exists

```
/cc-om:status
```

or `om status` from the session's Bash tool. Then read the listed files in the printed memory directory.

## Consolidate the ledger now

Consolidation runs on its own mid-session and at session end. To force it:

```
/cc-om:consolidate
```

or `om consolidate --cwd "$PWD"` from the session's Bash tool.

## Keep the compaction summary from duplicating the memory block

The persistent fix is the `CLAUDE.md` instruction from the [install step](install.md#recommended-add-the-compaction-instruction-to-claudemd) ([why](../explanation/design-decisions.md#accept-compaction-instead-of-controlling-it)); for a one-off manual compaction:

```
/compact do not reproduce the <observational-memory> block in the summary; it is re-injected automatically
```

## Correct a wrong memory

Edit the topic file directly, not while a consolidation is running ([copy-back](../reference/cli.md#manual-subcommands)). For contradictions the consolidator introduced, fix the entry and delete the stale "previously:" note.

## Reset memory

- **One session's ledger:** delete that session's directory under the data root (path shown by `om status`).
- **All durable memory for a project:** delete the memory directory shown by `om status`. This is irreversible.
