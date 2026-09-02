# Design decisions

The reasoning behind cc-om's contested choices. How the pieces fit together is in [architecture](architecture.md).

## Accept compaction instead of controlling it

Claude Code gives plugins no way to replace compaction or shape its summary; a `PreCompact` hook can only block it. cc-om therefore accepts it at whatever window the session runs: nothing in the design depends on when compaction happens, and the memory block is delivered by injection ([architecture](architecture.md#the-pipeline)) rather than by surviving in history. Losing the verbatim raw tail at compaction is accepted; observation density is the compensation.

The observer backlog is drained in the `PreCompact` hook, bounded by the hook timeout, a pass cap, and the minimum flush size ([CLI](../reference/cli.md#hook-subcommands)); whatever remains is picked up after compaction (mechanism: [the pipeline](architecture.md#the-pipeline)).

The compaction summary may preserve a stale copy of the previously injected block. Every block carries a freshness line marking older copies superseded, and a `CLAUDE.md` instruction can tell the summarizer to drop it ([install](../how-to/install.md#recommended-add-the-compaction-instruction-to-claudemd)).

## Memory ownership

Native auto memory and cc-om's consolidator would otherwise write the same directory, producing duplicate and contradictory entries, so the install settings give cc-om exclusive ownership (`autoMemoryEnabled: false`). A plugin cannot set this itself — plugin `settings.json` honors only `agent` and `subagentStatusLine` — so it is an install requirement. Because disabling auto memory also disables native index loading ([configuration](../reference/configuration.md#companion-settings-must-be-set-by-the-user)), cc-om injects the index itself. What is lost — the model's in-the-moment saves — is covered by the observer pipeline, which sees the whole transcript rather than what the model chose to save.

cc-om uses the same index/topic-file layout and default location as native auto memory (except in linked worktrees, [file layout](../reference/file-layout.md#durable-memory-directory)): inspectable and already familiar to Claude Code.

## The recall cliff, and pinned facts as the hedge

Cold topic files require the model to *decide* to read them. The assumption is that a coding model with a memory map will — it already lives in grep and file reads. The hedge is the pinned-facts tier: standing constraints are promoted to an always-injected block, so the worst case is losing detail, never invariants. User-pinned entries are exempt from the consolidator's cap-driven merging, so an explicit pin persists until the user removes it. The pinned-facts file also gives "current state" one canonical, frequently-rewritten home, limiting contamination by stale entries.

## No LLM on the interactive path

`SessionStart` and `UserPromptSubmit` hooks run synchronously before the user sees anything, so they do file reads and greps only. Observer calls run in the async `PostToolUse` hook and consolidation is detached; the one synchronous model path is the `PreCompact` drain, which runs while compaction waits. The retrieval assist is keyword-grep, not semantic; it can be replaced behind the same hook.

## Injection discipline

Appended injections do not bust the prompt cache, but every injected byte stays in history until compaction. Hence: the full memory block once per compaction epoch, a short per-prompt hint only when it matches, a capped pinned-facts file.

## Observer transport

Observer and consolidator calls shell out to `claude -p`. The observer transport is isolated in one function (`om_llm` in `bin/om`), so it can be replaced without touching the pipeline; the consolidator runs its own `claude -p` invocation because it is an agentic file-editing task, not a one-shot completion. Observer runs execute from the data root, so a `claude -p` observer does not ingest the project's own `CLAUDE.md` or memory context, and the transcript chunk is framed as data the observer must not obey. Nested sessions carry a marker that makes cc-om's own hooks exit immediately, so the plugin never observes its own model calls. Bookkeeping (chunking, watermarks, locks) is never delegated to a model: a failed observer call leaves the watermark so the chunk is retried, and consolidation works from a ledger snapshot so observer appends during a run are never lost.

## Transcript parsing is an accepted maintenance liability

The transcript JSONL is internal and version-unstable, and observers parse it anyway — it is the only source of raw history that survives compaction. The mitigation is defensive: unknown entries are dropped, failures leave the watermark so the chunk retries, and the extractor is isolated in one function (`extract_text` in `bin/om`).
