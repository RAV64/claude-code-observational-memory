# cc-om — Observational Memory for Claude Code

cc-om gives long-lived Claude Code sessions a memory that survives compaction. Background observers compress the session transcript into an observation ledger; a consolidator promotes aged observations into durable, grep-able topic files; hooks re-inject a memory block at every session start and after every compaction.

[Install](docs/how-to/install.md), then follow the [tutorial](docs/tutorials/getting-started.md).

## Documentation

| Section | Answers |
|---|---|
| [Tutorial: getting started](docs/tutorials/getting-started.md) | "teach me" |
| How-to: [install](docs/how-to/install.md), [configure](docs/how-to/configure.md), [manage memory](docs/how-to/manage-memory.md) | "how do I…?" |
| Reference: [CLI](docs/reference/cli.md), [configuration](docs/reference/configuration.md), [file layout](docs/reference/file-layout.md), [hooks](docs/reference/hooks.md) | "what exactly is…?" |
| Explanation: [architecture](docs/explanation/architecture.md), [design decisions](docs/explanation/design-decisions.md) | "why is it like this?" |

## Requirements

- Claude Code CLI (`claude`) on PATH
- `jq`
- bash, GNU coreutils, findutils, grep, sed, awk
- `flock` and `setsid` (util-linux)
- git (optional)
