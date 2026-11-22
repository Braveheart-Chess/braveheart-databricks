# Guidance for coding agents

This repository is a Databricks workspace project file,
[`jaws.yaml`](jaws.yaml), provisioned by
[jaws](https://github.com/jwallace145/jaws), and a Databricks asset bundle
([`databricks.yml`](databricks.yml)) of analysis notebooks, jobs and their tested Python package
(`src/selfplay_analysis/`). Read [README.md](README.md) for what the
project is and [CONTRIBUTING.md](CONTRIBUTING.md) for the full rules. The rules
below are the ones you MUST follow.

## Rules

- **Never push to `main`.** Every change goes through a pull request on a branch.
- **Commits are a single line**: `type(scope): subject`, at most 100 characters, with a lowercase imperative subject and no trailing period. Types: `feat`, `fix`, `perf`, `revert`, `docs`, `refactor`, `test`, `build`, `ci`, `style`, `chore`; never invent another. Scope: exactly one of `SCOPES` in `commitlint.config.mjs` (the Scopes table in CONTRIBUTING.md); never invent one, and if none fits, ask. The only thing allowed after the header is a blank line and `Co-authored-by: Name <email>` trailers. No body text. Put details in the PR description.
- **PR titles are valid commit headers**, scope included; squash merges use the title as the commit message.
- **PR bodies use the template's sections, in order, every one filled in**; write "n/a" rather than deleting one:

  ```sh
  gh pr create --title "type(scope): subject" --body-file <file>
  ```
- **Merges are squash or rebase only.** Never create merge commits.
- **Never apply or destroy the workspace yourself.** It is applied by the owner
  from jaws with the Databricks apply workflow. Do not run
  `jaws apply` or `destroy` against real resources.
- **Stay inside what the project file may ask for.** The network range, cost
  center, workspace admins and the allowed compute policies are set centrally in
  jaws's `config.yaml`. Change them there, not here.
- **`CHANGELOG.md` is Release Please's.** Never edit it by hand.
- **Keep secrets out of the repo.** The project file holds no credentials.
- **Never deploy the bundle's `prod` target yourself.** CI deploys it on merges to `main`, as the
  project's deployer. From a laptop, deploy the `dev` target only.
- **The GitHub environment `prod` and its variables are the owner's.** Never create or change them,
  and never commit a workspace host, application ID or account ID: the repository is public.

## Before opening a PR

```sh
npm ci
npm run test:commitlint   # the commit rules, the scopes table and the fixtures
npx commitlint --from origin/main --to HEAD --verbose
pre-commit run --all-files
uv sync && uv run pytest && uv run ruff check . && uv run ruff format --check .
```

## Things that need the owner

Repository settings and secrets, the release bot, running the Databricks apply
workflow, and adding a user to the Databricks account before the project file
lists them. List them in the PR instead of attempting them.
