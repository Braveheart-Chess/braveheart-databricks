# Contributing

This repository holds two things:

- [`jaws.yaml`](jaws.yaml), what this Databricks workspace
  project asks for. [jaws](https://github.com/jwallace145/jaws)
  provisions the workspace from it.
- A Databricks asset bundle, [`databricks.yml`](databricks.yml): analysis notebooks (`notebooks/`), the
  jobs that run them (`resources/`) and their tested Python package (`src/selfplay_analysis/`).

## Setup

```sh
npm ci                                                          # commitlint
uv sync                                                         # Python, for the bundle's package
pre-commit install --hook-type commit-msg --hook-type pre-commit
```

Before a pull request, run the tests and the linters:

```sh
uv run pytest                    # the Spark tests need a JDK 17 or 21; they skip without one, never in CI
uv run ruff check . && uv run ruff format --check .
```

The hooks are a convenience. CI is the authoritative check.

## Commit messages

Every commit is a **single-line** [Conventional Commit](https://www.conventionalcommits.org/) with a type and a scope:

```
type(scope): subject
```

- At most 100 characters, with an imperative, lowercase subject and no trailing period.
- `type` is one of `feat`, `fix`, `perf`, `revert`, `docs`, `refactor`, `test`, `build`, `ci`, `style`, `chore`: the same set in every repository.
- `scope` is required: exactly one of the scopes below. A change that spans two is two pull requests, or takes the scope it is mostly about.
- Breaking changes are marked with `!`, for example `feat(bundle)!: drop a compute policy`.
- The only content allowed after the header is a blank line followed by `Co-authored-by: Name <email>` trailers, in any letter case. No body text. Details belong in the PR description.
- The type drives the next version: `feat` bumps the minor version, `fix` the patch, and `!` the major (the minor while the version is below 1.0).

| Message | Valid |
| --- | --- |
| `feat(bundle): add a workspace user` | Yes |
| `fix(bundle): correct a policy name` + blank line + `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>` | Yes |
| `feat: add a workspace user` | No, no scope |
| `feat(nope): add a workspace user` | No, not one of this repository's scopes |
| `feat(bundle): add a workspace user` + blank line + `This adds…` | No, body text |

commitlint enforces this with `@commitlint/config-conventional` plus two local rules in `tools/commitlint/`: `only-coauthor-trailers` fails on any line after the header that is not blank or a `Co-authored-by` trailer, and `single-scope` fails on a list such as `feat(a,b)`. The fixtures in `tests/commitlint/` show accepted and rejected messages. CI lints every commit in the PR range, so a rebase merge never lands a non-conforming commit on `main`, and lints the PR title through the same config. The standard behind it is the [repository conventions](https://github.com/jwallace145/repository-template/blob/main/repository-conventions.md).

### Scopes

Every commit names exactly one of this repository's scopes. They are `SCOPES` in `commitlint.config.mjs`; this table mirrors it, in the same order, and `npm run test:commitlint` fails if they drift. Adding, renaming or removing a scope goes through a pull request like any other change.

| Scope | Covers |
| --- | --- |
| `analysis` | The `selfplay_analysis` package in `src/` and its tests |
| `notebooks` | The notebooks in `notebooks/` |
| `bundle` | `databricks.yml` and the jobs in `resources/` |
| `research` | The research write-ups in `docs/` |
| `deploy` | The Deploy workflow and `jaws.yaml` |
| `repo` | README, CONTRIBUTING, AGENTS, commitlint, the PR and issue templates, and repository tooling |
| `deps` | Dependency updates, mostly Dependabot's |
| `release` | Release Please's release pull requests and its config |

## Pull requests

Every change reaches `main` through a pull request. Nobody pushes to `main`.

- The title is a valid commit header, scope included: squash merges use it as the commit message. Release Please's own pull requests are titled `chore(release): release X.Y.Z`.
- Fill in every section of the one template, `.github/pull_request_template.md` (Summary, Why, Changes, Verification, Checklist). Write "n/a" rather than deleting one.
- Pull requests merge by squash or rebase only. There are no merge commits.
- Dependabot's titles are `chore(deps): bump ...` and pass as they are; squash merge them.

## Issues

Open issues from the forms: **Bug**, **Feature** or **Task**. Blank issues are off.
## How a change reaches the workspace

**The bundle** deploys when its change merges: the **Deploy** workflow signs in
as the project's deployer (GitHub OIDC, no secret) and runs
`databricks bundle deploy -t prod`. Nobody deploys `prod` by hand; deploy the
`dev` target from your laptop to try a change first (README, *Developing*).

**The project file** is applied by the infra repository, which reads it from
this repository's `main`:

1. Merge the change to `main`.
2. The next pull request in jaws shows it in its **Databricks plan** comment.
3. The owner runs the **Databricks apply** workflow in jaws for this project.

The network range, cost center, workspace admins and the compute policies the
project may use are set centrally, in jaws's `config.yaml`.
The fields of the project file are documented in
[docs/databricks.md](https://github.com/jwallace145/jaws/blob/main/docs/databricks.md#the-project-file).

## Releases

[Release Please](https://github.com/googleapis/release-please) keeps a release
pull request open with the next version and the changelog. Merging it tags the
version. It owns `CHANGELOG.md`; nobody edits that file by hand.
