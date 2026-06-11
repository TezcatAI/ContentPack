# Tezcat Official Content Pack

The first-party content pack for [Tezcat](https://github.com/TezcatAI/Tezcat), installed and enabled by default as a **verified official source**.

A *content pack* is a git repository with a `tezcat-pack.yaml` manifest plus one declarative YAML file per content item. Tezcat's package pipeline fetches, validates, and applies a pack into the running instance, see the manifest schema and pipeline in the Tezcat `apps/packages` subsystem.

## Layout

```
tezcat-pack.yaml      # the manifest (namespace, version, contents list)
prompts/*.yaml        # one capability-probe prompt per file
```

Every prompt here is stamped into the reserved `tezcat` namespace on install.

## Authoring your own pack

Clone [`TezcatAI/ContentPackTemplate`](https://github.com/TezcatAI/ContentPackTemplate) to start a pack in your own namespace, then add it as a source in the Tezcat Extensions hub.
