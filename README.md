# Tezcat Official Content Pack

The first-party content pack for [Tezcat](https://github.com/TezcatAI/Tezcat), installed and enabled by default as a **verified official source**.

A *content pack* is a git repository with a `tezcat-pack.yaml` manifest plus one declarative YAML file per content item. Tezcat's package pipeline fetches, validates, and applies a pack into the running instance, see the manifest schema and pipeline in the Tezcat `apps/packages` subsystem.

## Layout

```
tezcat-pack.yaml      # the manifest (namespace, version, contents list)
frameworks/*.yaml     # one risk framework per file (OWASP ASI, NIST AI RMF, AVID)
capabilities/*.yaml   # one typed capability per file, with framework mappings
prompts/*.yaml        # one capability-probe prompt per file
profiles/*.yaml       # one probe profile (a named set of prompts) per file
```

Every item here is stamped into the reserved `tezcat` namespace on install. The manifest lists the content types in dependency order (frameworks, then capabilities, then prompts, then profiles) so that each item's cross-references resolve as the pipeline applies them: capabilities reference framework items, prompts reference capability types, and profiles reference prompts.

## Releasing updates

### Version fields

Every content pack has two levels of versioning:

**Pack version** (`version` in `tezcat-pack.yaml`) tracks the pack as a whole. Tezcat displays this version in the Extensions hub and uses it in the update-available prompt.

**Content version** (`version` inside each `prompts/*.yaml`, `profiles/*.yaml`, etc.) forms the content coordinate `namespace:slug@version`. Changing this field creates a new logical item; the old version stays in the database so existing probe-run records remain valid.

Use [semantic versioning](https://semver.org) for both:

| Change | Pack version bump | Content version bump |
|---|---|---|
| Fix a typo or regex in an existing item | patch (`1.0.0 → 1.0.1`) | patch (`1.0.0 → 1.0.1`) |
| Add new prompts or profiles | minor (`1.0.0 → 1.1.0`) | — (new files start at `1.0.0`) |
| Rename slugs, remove items, or change the namespace | major (`1.0.0 → 2.0.0`) | major if slug kept |

### Release workflow

1. Edit content files under `frameworks/`, `capabilities/`, `prompts/`, or `profiles/`.
2. Bump the `version` field in any changed content file.
3. Bump `version` in `tezcat-pack.yaml` to reflect the scope of the change.
4. Commit and push to the repository.
5. In the Tezcat Extensions hub, click **Check** on the pack to detect the new version, then **Update** to apply it. All update types (patch, minor, major) require confirmation before applying.


## Prompts and prompt profiles

`prompts/*.yaml` are the capability probes: the text sent to a target agent plus the parser rules that turn its reply into a per-capability present/absent finding. Each prompt targets one or more capability types by slug and comes in one of two tiers:

- **Direct** (`*-direct`, tag `baseline`) asks the agent to self-report whether it has a capability. Low difficulty; a model can simply claim anything.
- **Behavioral** (`*-behavioral`, tag `behavioral`, ADR 0012 Layer 3) asks the agent to actually exercise the capability and return a computed sentinel, which is harder to fake. When the target capability declares `prerequisites`, Tezcat gates the probe: it is skipped in a run when a prerequisite was found absent.

`profiles/*.yaml` are named, runnable sets of prompts. A profile selects its prompts either dynamically by tag (`resolver_mode: filter_spec`, e.g. `baseline-capability-sweep` and `behavioral-escalation`) or as a pinned, ordered list of coordinates (`resolver_mode: explicit_list`, e.g. `all-probes`).

## Frameworks and capabilities

`frameworks/*.yaml` and `capabilities/*.yaml` are first-class pack content, listed in `tezcat-pack.yaml` and applied by their own content handlers.

**Frameworks** carry the curated risk taxonomies Tezcat maps findings against: OWASP Top 10 for Agentic Applications, NIST AI RMF, and AVID. Each file declares the framework metadata and its enumerated items (categories, controls, or taxonomy codes).

**Capabilities** are the typed capabilities Tezcat tracks (filesystem read, code execution, credential access, and so on). Each file carries a kebab-case slug, a default severity, and `framework_items` that link the capability to the framework codes it maps to. Prompts target these capability types by slug.

## Authoring your own pack

Clone [`TezcatAI/ContentPackTemplate`](https://github.com/TezcatAI/ContentPackTemplate) to start a pack in your own namespace, then add it as a source in the Tezcat Extensions hub.
