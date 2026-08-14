# goose-recipes

Reusable [goose](https://goose-docs.ai) recipes encoding a **spec-first development process**: turn a project proposal into validated design documents, an implementation plan, a generated repository, and finally execution of that plan.

The process is modeled on the [stripe-toddler](https://github.com/nickbrett1/stripe-toddler) build, which went through exactly this chain:

```
proposal → design docs → specdag validation → implementation plan → genproj scaffold → execute
```

---

## The recipes

| # | Recipe | Input → Output |
|---|--------|----------------|
| 1 | [`design-project.yaml`](design-project.yaml) | Proposal (or clarifying-questions interview) → `description.md` + full `spec/` artifact tree |
| 2 | [`validate-spec.yaml`](validate-spec.yaml) | `description.md` + `spec/` → `spec/dependency-map.yaml`, LLM traceability audit, specdag checks, `_generated/` report |
| 3 | [`generate-plan.yaml`](generate-plan.yaml) | Validated spec → `plan.md` (phased, `[MANUAL]`/`[AGENT]`-tagged, tooling per step) |
| 4 | [`scaffold-project.yaml`](scaffold-project.yaml) | `plan.md` → genproj creates the repo, design docs copied in, **pauses for you to clone** |
| 5 | [`execute-plan.yaml`](execute-plan.yaml) | `plan.md` → executes the plan phase-by-phase from inside the generated project |

Recipes 1–5 are the spec-first chain: run them in order. Each recipe validates that its inputs exist and stops with a clear message if a previous step is missing.

### NAS operations

| # | Recipe | Input → Output |
|---|--------|----------------|
| 6 | [`add-nas-container.yaml`](add-nas-container.yaml) | New container name/image/params → `/volume1/docker/<name>/compose.yaml` (sensible mem limits + correct watchtower labels), gethomepage entry, deployed & memory-verified |

NAS-specific (not part of the spec-first chain). Run from a goose session on the NAS; it models the nas-port-mcp / parquet-peek additions: composes the stack with `mem_limit`/`memswap_limit` caps, routes your own `ghcr.io/nickbrett1/*` images to watchtower-nick (`scope=nick`, 60s poll) via labels and third-party images to the plain nightly watchtower, then adds the tile to `/volume1/docker/homepage/config/services.yaml` and verifies memory/health post-deploy.

## The contract (shared paths)

All recipes agree on one layout. `{{ project_root }}` is the parameter you pass when running a recipe.

```
<project_root>/
├── description.md                # proposal / business case (recipe 1)
├── spec/
│   ├── topology/                 # component-graph.mmd, deployment-targets.md
│   ├── api/                      # <service>-openapi.yaml, <client>-protocols.md
│   ├── data-architecture/        # <store>-schema.sql, kv-layout.json, domain-models.<lang>
│   ├── flows/                    # <flow>-sequence.md, <flow>-state-machine.md
│   ├── ui/                       # design-system.md, wireframes/*.excalidraw
│   ├── capacity/                 # load-model.md
│   ├── event-flow.md             # event schemas
│   ├── implementation-considerations.md   # constraint ledger (read carefully by recipe 3)
│   └── dependency-map.yaml       # ESDD/SpecDAG graph (recipe 2)
├── _generated/                   # specdag output: assembled-map.json, report.html (recipe 2)
└── plan.md                       # implementation plan (recipe 3)
```

When recipe 4 copies the design docs into the generated repo, they land in `<repo>/specs/`.

## Installation

Recipes are global. Two options:

**Option A — GitHub repo (recommended).** Point goose at this repo directly:

```bash
export GOOSE_RECIPE_GITHUB_REPO="nickbrett1/goose-recipes"
```

Requires the GitHub CLI (`gh`) installed and authenticated.

**Option B — clone into the global recipes directory.**

```bash
git clone https://github.com/nickbrett1/goose-recipes.git ~/.config/goose/recipes/
```

(Alternatively add the repo path via `GOOSE_RECIPE_PATH`.)

Verify discovery with `goose recipe list`.

## Usage

Run the recipes in order from a goose session in the workspace where you want the design artifacts to live:

1. **Design** — `/recipe design-project` (provide `project_name`, `project_root`; optionally a `proposal_file` and a `conventions_file`)
2. **Validate** — `/recipe validate-spec` (provide `project_root`)
3. **Plan** — `/recipe generate-plan` (provide `project_root`)
4. **Scaffold** — `/recipe scaffold-project` (provide `project_root`, `project_name`, optional `repository_url`/`capabilities`) — this recipe **pauses** for you to clone the generated repo
5. **Execute** — launch a new goose session inside the cloned project (its devcontainer), then `/recipe execute-plan`

For NAS container ops (Synology `/volume1/docker` stacks): `/recipe add-nas-container` (provide `container_name`, `image`, `homepage_group`; optionally `is_own_container`, `ports`, `mem_limit`, `volumes`, `environment`, `icon`, `health_endpoint`, `widget`).

## Prerequisites

- **goose CLI** (recipes run in the CLI or Desktop)
- **`fintechnick` MCP extension** enabled (used by `scaffold-project` for genproj). Requires the `FINTECHNICK_MCP` environment variable.
- **`specdag` CLI** installed and on `PATH` (used by `validate-spec`)
- A GitHub account + credentials for genproj repo creation and cloning

## Conventions

The default development conventions are in [`conventions/development-conventions.md`](conventions/development-conventions.md) (derived from the ftn project constitution): mandatory tests with >80% coverage, strict lint/format, secrets via Doppler when any exist, CircleCI pipelines, WCAG 2.1 AA for UI surfaces, Lighthouse ≥90 when configured, and ADRs for deviations.

**Overriding:** pass a `conventions_file` parameter to `design-project` to bind a different conventions document for that project (e.g. `conventions/development-conventions.md` from this repo, or a project-specific fork). Recipe 1 will treat the provided file as binding; deviations are documented in ADRs.
