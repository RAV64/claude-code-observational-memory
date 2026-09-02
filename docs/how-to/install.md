# How to install cc-om

## For development (local checkout)

```bash
claude --plugin-dir /path/to/cc-om
```

Reload after editing plugin files with `/reload-plugins` inside the session.

## Required: set the companion settings

Set cc-om's two operating assumptions in the project's `.claude/settings.json`:

```json
{
  "autoMemoryEnabled": false,
  "autoCompactWindow": 300000
}
```

Rationale: [design decisions](../explanation/design-decisions.md#memory-ownership), [compaction](../explanation/design-decisions.md#postpone-compaction-instead-of-controlling-it).

User-level `~/.claude/settings.json` works too and covers every project at once; use per-project settings only if some repos should keep native auto memory. Unless auto memory is disabled, `om session-start` warns ([what is checked](../reference/cli.md#hook-subcommands)).

## Recommended: add the compaction instruction to `CLAUDE.md`

Add to the project's `CLAUDE.md` ([why](../explanation/design-decisions.md#postpone-compaction-instead-of-controlling-it)):

```markdown
When compacting this conversation, do not reproduce the `<observational-memory>` block in the summary — it is maintained by the cc-om plugin and re-injected automatically after compaction.
```

## As a persistent plugin

1. Publish the repo (or a fork) with a `.claude-plugin/marketplace.json` index, then:

   ```bash
   claude plugin marketplace add <your-marketplace-repo-url>
   claude plugin install cc-om@<marketplace-name>
   ```

2. Verify it is active: run `/cc-om:status` in a session.

## Validate before distributing

```bash
claude plugin validate --strict /path/to/cc-om
```

Fix any reported manifest or hook-schema issues before publishing.

## Disable or remove

```bash
claude plugin disable cc-om     # keep installed, stop hooks
claude plugin uninstall cc-om
```

Durable memory files are not removed by uninstalling; see [manage memory](manage-memory.md) to delete them. Session state under the data root is deleted with the plugin unless you pass `--keep-data`.
