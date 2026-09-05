# Railway Repository Template

Starter repository and shared Knitto template for Railway
infrastructure-as-code repositories.

The repository root is immediately usable and includes:

- `package.json` with the Railway TypeScript IaC and quality toolchain.
- `tsconfig.json` for strict NodeNext `.railway/**/*.ts` code.
- `.railway/railway.ts` with an empty project named from the checkout directory.
- `.railway/docker-images.json` for project-specific pinned image metadata.
- `.env.example` for non-secret project inputs.
- `.gitignore` for local secrets, dependencies, and account-specific plans.
- Pull request quality checks for Prettier, package ordering, and TypeScript.
- A main-branch repair workflow that opens a pull request for safe automatic
  formatting and package ordering fixes.
- A manually dispatched workflow that applies the latest Knitto
  template revision and opens a pull request with the resulting changes.

`.knitto` contains the ongoing shared policy for files that should remain
consistent across Railway repositories. The Railway resource graph itself is
seeded from this repository but becomes project-owned because each application
has different services, databases, buckets, volumes, and variables.

GitHub copies `.knitto` into repositories created from this template so
the complete starter state remains inspectable. The copied
`.knitto.json` initially uses that local embedded source.
Template-defined required inputs collect the generated repository's package
name and description whenever the selected template revision needs them.
Node.js support remains shared policy and is fixed at version 22 or newer.
After those values exist, the first plan rewrites `.knitto.json` to the
canonical Git source and removes the embedded `.knitto` copy. The
`template-railway` parent repository is explicitly excluded from its identity
prompts, package reconciliation, source rewrite, and embedded-template
deletion, so `knitto plan --update` can update the template itself
without changing the starter state copied to children.

The parent still dogfoods its own policy. Its root `.knitto.json` points to the
embedded `.knitto` source, and the template is ready to propagate only when:

```bash
knitto check --update
```

reports zero drift and zero failed checks. Parent-specific behavior must be
expressed through explicit template conditions rather than unmanaged
differences. The managed pull-request quality workflow runs this command for
the template parent and every generated consumer.

## Releases

This template opts into immutable releases through Release Please. Development
continues on `main`; the Release Please pull request updates `package.json`,
`package-lock.json`, `.release-please-manifest.json`, and
`.knitto/template.json`. Merging that pull request creates the template's
configured `v{version}` tag.

Before the first release, Release Please's `0.0.0` bootstrap version is not a
real tag. Consumers use `main` during this bootstrap period. After the first
release, their generated `.knitto.json` pins the exact release tag and the exact
public `knitto` npm version required by that template:

```json
{
  "source": {
    "type": "git",
    "url": "https://github.com/reggi/template-railway.git",
    "path": ".knitto",
    "ref": "v0.0.1"
  },
  "engine": {
    "package": "knitto",
    "version": "0.0.1"
  }
}
```

Conductor dispatches the update workflow with the new immutable tag. The
workflow first runs `knitto source pin --ref <tag>` with the currently pinned
engine, then runs `knitto apply --update` with the newly required engine.
Reverting that pull request restores the prior template and engine pins.

Templates that do not want release tags can omit the `release` block and keep
using a branch ref exactly as before.

## Create a repository

Create a new GitHub repository from `reggi/template-railway`, then clone the
generated repository. The root files provide its complete initial state, but
`package.json` intentionally has no `name` or `description`. The template
explicitly offers the checkout directory name as the default for
`metadata.name`; the user may accept it or enter a different package name.
From inside the checkout, run a plan:

```bash
knitto plan
knitto apply
```

The plan detects that `metadata.name` and `metadata.description` are missing,
prompts with any template-configured defaults, saves the answers to
`.knitto.json`, and then shows
the enforced `package.json` changes. `.railway/railway.ts` reads the resulting
explicit package name rather than deriving identity from `process.cwd()`. The
first apply switches the project source to
`https://github.com/reggi/template-railway.git`, removes the copied
`.knitto` directory, and creates the lock. Later template revisions may
introduce additional required inputs; the next `plan --update` resolves them in
the same way. The project-owned `.railway/railway.ts` starter is not replaced.

Automation can inspect the template requirements without triggering prompts:

```bash
knitto inputs --update --json
```

Preview and apply later template revisions from the managed repository:

```bash
knitto plan --update
knitto apply --update
```

After package dependency changes are applied, refresh the generated lockfile:

```bash
npm install --package-lock-only --ignore-scripts
```

The repair workflow needs the repository setting that allows GitHub Actions to
create pull requests. TypeScript errors fail the workflow and require a human
fix; only deterministic Prettier and `sort-package-json` changes are committed
automatically.

The **Update repository template** workflow can be started from the Actions
tab. It builds Knitto from `reggi/knitto`, runs
`apply --update`, refreshes `package-lock.json` when needed, and opens or
updates a pull request. Both pull-request-producing workflows require GitHub
Actions to be allowed to create pull requests in the repository settings.

The optional `set` workflow input accepts a JSON object whose keys are
Knitto input paths and whose values are strings, numbers, or
booleans:

```bash
gh workflow run update-template.yml \
  --repo reggi/railway-vikunja \
  --ref main \
  -f 'set={"metadata.name":"railway-vikunja","metadata.description":"Railway infrastructure for Vikunja"}'
```

Each entry is passed safely as a separate `--set path=value` argument. Invalid
JSON, arrays, objects, and null values fail before Knitto runs.

`apply --update` plans and applies in the same CI step. If a newer template
introduces an input that has not been configured, CI stops instead of choosing
an answer. Its error explains how to run `knitto plan --update`
interactively or with explicit `--set` values, then commit the resulting
`.knitto.json` through a manual pull request.
