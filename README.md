# Tezcat Official Content Pack

The first-party content pack for [Tezcat](https://github.com/TezcatAI/Tezcat), installed and enabled by default as a **verified official source**.

A *content pack* is a git repository with a `tezcat-pack.yaml` manifest plus one declarative YAML file per content item. Tezcat's package pipeline fetches, validates, and applies a pack into the running instance, see the manifest schema and pipeline in the Tezcat `apps/packages` subsystem.

## Layout

```
tezcat-pack.yaml      # the manifest (namespace, version, contents list)
prompts/*.yaml        # one capability-probe prompt per file
```

Every prompt here is stamped into the reserved `tezcat` namespace on install.

## Releasing updates

### Version fields

Every content pack has two levels of versioning:

**Pack version** (`version` in `tezcat-pack.yaml`) — tracks the pack as a whole. Tezcat displays this version in the Extensions hub and uses it in the update-available prompt.

**Content version** (`version` inside each `prompts/*.yaml`, `profiles/*.yaml`, etc.) — forms the content coordinate `namespace:slug@version`. Changing this field creates a new logical item; the old version stays in the database so existing probe-run records remain valid.

Use [semantic versioning](https://semver.org) for both:

| Change | Pack version bump | Content version bump |
|---|---|---|
| Fix a typo or regex in an existing item | patch (`1.0.0 → 1.0.1`) | patch (`1.0.0 → 1.0.1`) |
| Add new prompts or profiles | minor (`1.0.0 → 1.1.0`) | — (new files start at `1.0.0`) |
| Rename slugs, remove items, or change the namespace | major (`1.0.0 → 2.0.0`) | major if slug kept |

### Release workflow

1. Edit content files under `prompts/` or `profiles/`.
2. Bump the `version` field in any changed content file.
3. Bump `version` in `tezcat-pack.yaml` to reflect the scope of the change.
4. Commit and push to the repository.
5. In the Tezcat Extensions hub, click **Check** on the pack to detect the new version, then **Update** to apply it. All update types (patch, minor, major) require confirmation before applying.

### Force reimport vs Update

| Action | What it does |
|---|---|
| **Check** | Fetches the remote repo and reports whether a newer version is available. Makes no changes to the catalog. |
| **Update** | Pulls the latest commit from the remote, then runs the full pipeline (validate → apply). Requires confirmation. |
| **Force reimport** | Re-applies the pack from the copy already on disk — no network call, no git pull. Useful after editing submodule files directly or after a failed apply that left the catalog in a partial state. |

## Authoring your own pack

Clone [`TezcatAI/ContentPackTemplate`](https://github.com/TezcatAI/ContentPackTemplate) to start a pack in your own namespace, then add it as a source in the Tezcat Extensions hub.
