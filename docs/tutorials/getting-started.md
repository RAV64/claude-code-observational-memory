# Getting started with cc-om

In this tutorial you will run your first Claude Code session with observational memory, pin a fact, inspect the memory state, and see the memory block survive a context reset.

You need the [requirements](../../README.md#requirements) installed, a local checkout of this plugin, and the [companion settings](../how-to/install.md#required-set-the-companion-setting) set for your project or user.

## 1. Start a session with the plugin loaded

From any project directory, run:

```bash
claude --plugin-dir /path/to/cc-om
```

Claude Code starts normally. cc-om is now listening to the session through its hooks.

## 2. Pin your first fact

In the session, type:

```
/cc-om:remember this project targets PostgreSQL 18, never suggest MySQL syntax
```

Claude rewrites the fact into a self-contained bullet and confirms what it pinned. This fact now lives in `pinned-facts.md` and will be injected into every future session of this project until you remove it.

## 3. Do some real work

Ask Claude to explore the project, or fix something small — anything that produces a few minutes of activity. Short sessions may not trigger the observer yet; that is expected.

## 4. Look at the memory state

Type:

```
/cc-om:status
```

You will see the memory directory for this project and, per session, the observation ledger size and the watermark showing how much transcript the observer has processed.

## 5. See the memory survive a reset

Now type:

```
/clear
```

Your conversation history is gone. Ask:

```
What database does this project target?
```

Claude answers PostgreSQL 18 — not from the conversation (which was just erased), but from the pinned-facts block cc-om injected when the cleared session started. To see the file behind it, ask Claude to read `pinned-facts.md` from the memory directory `/cc-om:status` printed in step 4.

## Where to go next

- Longer sessions will fill the observation ledger; [manage memory](../how-to/manage-memory.md) shows consolidation and inspection tasks.
- To understand what just happened, read the [architecture explanation](../explanation/architecture.md).
