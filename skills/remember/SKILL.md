---
description: Pin a fact into the always-injected pinned-facts memory block so it can never fall out of passive recall. Use when the user says something must always be remembered.
allowed-tools: Bash, Read
---

Pin the fact given in $ARGUMENTS:

1. Rewrite it as a single, self-contained bullet: concrete, present tense, understandable without this conversation.
2. Run `om remember "<rewritten fact>"` from the project root (`"${CLAUDE_PLUGIN_ROOT}/bin/om"` if `om` is not on PATH).
3. Read `pinned-facts.md` in the project memory directory. If it now exceeds 20 bullets, tell the user which existing bullets look least critical and ask whether to merge or evict them - do not evict on your own.

Confirm to the user what was pinned, quoting the exact bullet.
