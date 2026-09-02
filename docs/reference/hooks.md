# Hook wiring reference

Defined in `hooks/hooks.json`, which Claude Code loads by convention. All handlers are `type: command` invocations of `${CLAUDE_PLUGIN_ROOT}/bin/om`; exit behavior is in the [CLI reference](cli.md#exit-behavior).

| Event | Matcher | Subcommand | Timeout | Async |
|---|---|---|---|---|
| `SessionStart` | `startup\|resume\|clear\|compact\|fork` | `om session-start` | 10 s | no |
| `UserPromptSubmit` | (all) | `om user-prompt` | 5 s | no |
| `PostToolUse` | `.*` | `om observe` | none (not enforced for async hooks; the observer call is capped, [CLI](cli.md#hook-subcommands)) | yes |
| `PreCompact` | (all) | `om pre-compact` | 600 s | no |
| `SessionEnd` | (all) | `om session-end` | 1.5 s (SessionEnd budget; plugin hook timeouts do not raise it) | no |

`om finalize`, detached by `om session-end`, is not bounded by the hook budget.

Rationale for the synchronous/async split: [design decisions](../explanation/design-decisions.md#no-llm-on-the-interactive-path).
