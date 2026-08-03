# goose-recipes

Reusable [goose](https://goose-docs.ai) recipes encoding a **spec-first development process**: turn a project proposal into validated design documents, an implementation plan, a generated repository, and finally execution of that plan.

The process is modeled on the [stripe-toddler](https://github.com/nickbrett1/stripe-toddler) build, which went through exactly this chain:

```
proposal → design docs → specdag validation → implementation plan → genproj scaffold → execute
```

## The five recipes

| # | Recipe | Input → Output |
|---|--------|----------------|
| 1 | [`design-project.yaml`](design-project.yaml) | Proposal (or clarifying-questions interview) → `description.md` + full `spec/` artifact tree |
| 2 | [`validate-spec.yaml`](validate-spec.yaml) | `description.md` + `spec/` → `spec/dependency-map.yaml`, LLM traceability audit, specdag checks, `_generated/` report |
| 3 | [`generate-plan.yaml`](generate-plan.yaml) | Validated spec → `plan.md` (phased, `[MANUAL]`/`[AGENT]`-tagged, tooling per step) |
| 4 | [`scaffold-project.yaml`](scaffold-project.yaml) | `plan.md` → genproj creates the repo, design docs copied in, **pauses for you to clone** |
| 5 | [`execute-plan.yaml`](execute-plan.yaml) | `plan.md` → executes the plan phase-by-phase from inside the generated project |

Run them in order. Each recipe validates that its inputs exist and stops with a clear message if a previous step is missing.

## Installation

**Option A — GitHub repo (recommended).** Point goose at this repo directly:

```bash
export GOOSE_RECIPE_GITHUB_REPO="nickbrett1/goose-recipes"
```

Requires the GitHub CLI (`gh`) installed and authenticated.

**Option B — clone into the global recipes directory.**

```bash
git clone https://github.com/nickbrett1/goose-recipes.git ~/.config/goose/recipes/
```

Verify discovery with `goose recipe list`.

## Usage

1. **Design** — `/recipe design-project` (provide `project_name`, `project_root`; optionally `proposal_file` and `conventions_file`)
2. **Validate** — `/recipe validate-spec` (provide `project_root`)
3. **Plan** — `/recipe generate-plan` (provide `project_root`)
4. **Scaffold** — `/recipe scaffold-project` (provide `project_root`, `project_name`, optional `repository_url`/`capabilities`) — **pauses** for you to clone the generated repo
5. **Execute** — launch a new goose session inside the cloned project (its devcontainer), then `/recipe execute-plan`

## Prerequisites

- goose CLI
- `fintechnick` MCP extension enabled (used by `scaffold-project` for genproj) with `FINTECHNICK_MCP` env var
- `specdag` CLI on `PATH` (used by `validate-spec`)
- GitHub credentials for genproj repo creation and cloning

## Conventions

Default development conventions: [`conventions/development-conventions.md`](conventions/development-conventions.md) (derived from the ftn constitution). Override per project by passing a `conventions_file` to `design-project`.

See the full contract (shared paths) in the README history / repo docs.
