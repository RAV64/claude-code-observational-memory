# How to configure cc-om

cc-om's tunables are environment variables read by `bin/om`; the full list with defaults is in the [configuration reference](../reference/configuration.md). Set them in the environment Claude Code starts from, or in the `env` block of a settings file.

## Change the consolidator model

```bash
export OM_CONSOLIDATOR_MODEL=opus
```

`OM_OBSERVER_MODEL` selects the observer model the same way.

## Make observers run more or less often

Observers wake when this many new transcript bytes have accumulated:

```bash
export OM_CHUNK_BYTES=30000
```

## Compact earlier than the model's limit

cc-om does not need this; it is a cost knob. By default Claude Code compacts at the model's context window (1M tokens on current models). A smaller window makes turns cheaper and refreshes the memory block sooner, at the price of more frequent compaction. Set it per project in `.claude/settings.json`:

```json
{ "autoCompactWindow": 300000 }
```

The value cannot exceed the model's context window ([reference](../reference/configuration.md#related-claude-code-settings)).

## Provide credentials to hook-spawned observers

If observer runs fail to authenticate, put the credential in the env file `om` sources at startup (`OM_ENV_FILE`), and restrict its permissions:

```bash
mkdir -p ~/.config/cc-om
install -m 600 /dev/null ~/.config/cc-om/env
# then add: export CLAUDE_CODE_OAUTH_TOKEN=...
```

## Relocate the memory directory

To keep memory somewhere other than the [default location](../reference/file-layout.md#durable-memory-directory) (for example, inside the repo), set `OM_MEMORY_DIR` for that project only, in the `env` block of its `.claude/settings.json`:

```json
{ "env": { "OM_MEMORY_DIR": "/path/to/repo/.memory" } }
```
