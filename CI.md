# What the CI workflows have in common

Every public repository here builds something different — Swift, Python, Node,
shell, a Homebrew tap — so the *commands* in CI are never going to match. What
should match is the frame around them: the triggers, the permissions, the
timeouts and the hygiene jobs. Those have nothing to do with the language, and
a repository that quietly omits one is not simpler, it is just less protected.

This file is that frame. `pfadi`, `git-tidy` and `browser-in-a-box` are the
worked examples; when this document and those repositories disagree, they are
the ones that ran.

## There is no pre-commit framework, on purpose

The checks that decide whether a change lands are the ones in
`.github/workflows/`. There is no `.pre-commit-config.yaml`, no husky, no
lint-staged, and no second list of the same commands in a hook config.

A hook config is a copy of CI that nobody re-runs when CI changes. It goes out
of step within a month, and then it fails locally on rules CI does not have, or
passes locally on rules CI does. It also cannot be trusted for anything that
matters, because a contributor can skip it with `--no-verify` and a fork never
had it installed at all.

Run the commands from the README's **Development** section before you push. CI
runs the same ones, and CI is the one that counts.

## Every workflow

**File and name.** One concern per file: `ci.yml`, `release.yml`,
`codeql.yml`. The `name:` is lower case and matches the filename — `ci`, not
`CI`. Job ids are lower case too, and a job that is not obvious carries a
`name:` in words: `name: lint workflows`.

**Triggers, in this order.**

```yaml
on:
  pull_request:
  push:
    branches: [main]
  workflow_dispatch:
```

`workflow_dispatch` is not optional. A pull request opened by a workflow using
`GITHUB_TOKEN` does not trigger `pull_request` runs, and dispatch is the
documented way to run the required checks on such a branch anyway. Add a weekly
`schedule:` where an upstream release can break the build without anyone
pushing — a new Swift, a new base image, a rebuilt upstream container.

**Least privilege at the top of the file.**

```yaml
permissions:
  contents: read
```

Always present, always at workflow level, always read-only there. A job that
genuinely needs to write says so in its own `permissions:` block, and only for
what it writes. A missing top-level block does not mean "no permissions", it
means whatever the repository default happens to be, which is not a decision
anybody made.

**Concurrency.**

```yaml
concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true
```

Superseded runs on the same ref are wasted minutes. The exception is any job
that writes — a release, a tap update: same group, but
`cancel-in-progress: false`, because a run cancelled between writing and pushing
leaves a mess that nobody asked for.

**A timeout on every job.** `timeout-minutes:` is per job, and a job without one
inherits GitHub's six-hour default; a hung test then holds the queue for the
rest of the afternoon. Ten minutes for lint, twenty for a build, and raise it
when a real run gets close rather than pre-emptively.

**No `---` document start.** GitHub's own workflow and issue-form examples do
not carry one, and neither do most of these repositories. The yamllint config
disables the rule rather than leaving it at a warning nobody acts on.

**Action versions.** Major-version tags, one version per action across all
repositories: `actions/checkout@v7`, `actions/setup-node@v7`,
`actions/upload-artifact@v7`. Third-party actions are pinned more tightly
(`raven-actions/actionlint@v2.2.0`), because their major tag is not a promise
from anybody in particular. Dependabot bumps them; that is what the
`github-actions` ecosystem entry is for.

**`persist-credentials: false` on checkout** in any job that runs repository
code without needing to push. `actions/checkout` writes the job token into
`.git/config` by default, and `pull_request` runs code from strangers.

## The jobs a repository should have

Whatever the language, four things get checked. How is up to the repository;
whether is not.

1. **Format and lint.** Non-negotiable and separate from the tests, so a
   formatting failure reads as a formatting failure. Run the formatter in check
   mode, never in write mode, in CI.

   Where a repository has one script that runs everything — `npm run gate`,
   `make check` — that script stays the local one-word command, and CI still
   lists the steps individually. Each check is defined once either way; what
   the workflow adds is a checks list where the red one names itself instead of
   being a log to go and read. The release workflow is the exception and calls
   the whole script, because there is nothing to diagnose at a glance there.
2. **Tests, with a floor under the coverage.** A coverage number that is
   reported but not enforced is a number that goes down. The floor lives in the
   repository (a script, a config, a threshold flag), not in the workflow.
3. **The workflows themselves.** `actionlint`, `yamllint` and
   `markdownlint-cli2`, in one job called `workflows`. It costs about a minute
   and catches the class of mistake that is otherwise found by pushing.

   Two settings on the actionlint step, the same in every repository, because a
   linter configured differently per repository is not one linter:

   ```yaml
   env:
     # Advisory below a warning: SC2016 fires on every `${{ }}` inside a
     # single-quoted shell string, and nobody was going to act on those.
     SHELLCHECK_OPTS: --severity=warning
   with:
     # Only where a workflow uses `concurrency.queue`, which is a documented
     # key that actionlint's schema does not know. One token, no spaces: the
     # action hands `flags` to actionlint without a shell parsing it, so a
     # quoted pattern arrives split into pieces.
     flags: -ignore unexpected.key.+queue
   ```

4. **A secret scan.** `gitleaks`, with `fetch-depth: 0` so it sees the history
   and not just the tip. Several of these projects handle credentials for real
   accounts; this is the cheapest guard there is against one ending up in a
   fixture.

Repository-specific gates sit alongside those, not instead of them: a smoke
test that starts the MCP server and validates its tool schemas offline, a check
that the README's mermaid diagrams still parse, a packaging step that proves the
artefact is launchable.

**Soft gates should be rare and labelled.** `continue-on-error: true` turns a
check into a notification. It is defensible for `npm audit`, where the finding
is usually in a transitive dependency with no reachable path, and it needs a
comment saying why. Anywhere else, either the check matters and blocks, or it
does not belong in CI.

## Release workflows

Same frame, plus:

- A `guard` job first, which fails fast when the tag, the version in the
  manifest and the changelog entry disagree. Everything else `needs:` it.
- Build jobs per platform, then one `publish` job that consumes their artefacts.
- `permissions:` raised only inside the jobs that publish.
- `concurrency` with `cancel-in-progress: false`.
- Anything pushed to another repository (the Homebrew tap) is pulled by that
  repository on a schedule rather than pushed from here. A workflow may write to
  its own repository with the token it is handed, so the tap updating itself
  needs no secret at all; the alternative is a personal access token in every
  project, each of which expires separately.

## What is deliberately not standardised

The build commands, the runner OS, the language version, the test framework,
the schedule cadence, and how the coverage floor is enforced. Those follow the
project. Trying to unify them is how a shared workflow ends up with six inputs
and a conditional per repository.
