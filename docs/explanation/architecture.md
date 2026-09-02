# Architecture

This page explains how cc-om is put together and why each piece exists. For file paths and exact formats, see the [file layout](../reference/file-layout.md); for the reasoning behind contested choices, see [design decisions](design-decisions.md).

## The problem

Claude Code's native answer to a full context window is compaction: a one-shot, lossy summary. Facts learned two compactions ago are absent from context. cc-om's job is to make knowledge survive that boundary, and to keep accumulating it across sessions and days.

## Three temperatures

cc-om organizes memory by how it is recalled, not just by age:

```
HOT   — always in context, passive recall
        pinned-facts.md, journey tail, memory map, recent observations

WARM  — on disk in session state, not in context
        the observation ledger: atomic facts distilled from the transcript;
        its tail is HOT, its overflow is promoted to COLD

COLD  — on disk, active recall
        topic files (architecture.md, decisions.md, ...), grep/read-able,
        navigated via the memory map
```

The hot tier holds what must never be forgotten; the cold tier holds unbounded detail; the memory map is the bridge that tells the model when to descend ([why](design-decisions.md#the-recall-cliff-and-pinned-facts-as-the-hedge)).

## The pipeline

```
transcript JSONL (on disk, survives compaction)
      │  watermark-chunked (OM_CHUNK_BYTES)
      ▼
observer (OM_OBSERVER_MODEL, async PostToolUse hook, lock-guarded)
      │  atomic observation bullets
      ▼
observation ledger (session state)
      │  overflow at the ledger soft limit; whole ledger at session end
      ▼
consolidator (OM_CONSOLIDATOR_MODEL, detached)
      │  sorts by topic, reconciles contradictions
      ▼
topic files + MEMORY.md index + JOURNEY.md + pinned-facts promotions
      │
      ▼
SessionStart / SessionStart(compact) injection ──► back into context
```

Two properties are central:

**The source of truth is the transcript file, not the context window.** Compaction rewrites what the model sees; it does not touch the JSONL on disk. Observers work from the file, so they can lag, run async, and catch up after compaction; the `PreCompact` drain reduces the lag before the block is rebuilt.

**Memory delivery is injection, not history.** The memory block is re-injected after every compaction and at every session start; it never depends on surviving inside the conversation. Compaction becomes a survivable reset.

## Cache behavior

Between compactions the context is strictly append-only: observers work off to the side, injections land at the tail, and nothing rewrites already-sent history, so prompt-cache hits are preserved. The one full cache bust is compaction itself, which Claude Code pays with or without cc-om; how often it happens is set by the session's auto-compaction window ([configuration](../reference/configuration.md#related-claude-code-settings)), which cc-om leaves alone.
