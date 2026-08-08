# Contributing

This file is the default for every public repository under
[github.com/sapn95](https://github.com/sapn95). A repository that needs
something different ships its own `CONTRIBUTING.md`, and that one wins.

## Anyone may open a pull request

A GitHub account is the whole requirement.

Fork the repository, push a branch to your fork, open a pull request against
`main`. You do not need to be invited, added as a collaborator, or ask first.
There is no CLA to sign, no issue to be assigned, and no rule against picking up
something somebody else opened an issue about.

Nobody pushes to `main` in these repositories, including me. Every change
arrives as a pull request and runs the same checks as every other one.

## Open an issue first only for large changes

A bug fix, a failing edge case, a missing test, a clearer error message, a typo:
send the pull request. No preamble needed.

Anything that adds a dependency, changes a public interface, or takes more than
an evening: open an issue and say what you intend to do. These are spare-time
projects with one maintainer, and an issue costs you ten minutes where a
finished branch can cost you a weekend and still not be the direction the
project is going.

## What a good change looks like

- **One change per pull request.** A fix and a refactor on one branch is two
  reviews wearing one hat, and the fix is the one that waits.
- **Follow the repository, not your habits.** Formatting, naming and file
  layout come from the code around your change. Every repository has a
  formatter wired into CI; run it rather than argue with it.
- **Bring a test that fails without your fix.** Not a test that passes either
  way. If the thing you fixed cannot be reached from a test, say so in the pull
  request and explain how you verified it instead.
- **Comments explain why, not what.** The code already says what it does. The
  comment is for the reason it does it that way, and especially for the obvious
  alternative that does not work.
- **Never commit real data.** Several of these projects talk to Swiss postal,
  billing and tax portals. Fixtures are synthetic and named `Example_*`. No
  real account numbers, addresses, names, tokens or cookies — not in code, not
  in a test, not in a screenshot, and not in an issue.
- **New runtime dependencies need a sentence of justification** in the pull
  request. Most of these projects deliberately run on the standard library.
- **No telemetry, no analytics, no phone-home.** This is not negotiable in any
  of these repositories, however anonymous the payload.

## Commits

Look at `git log` in the repository you are changing and write subjects like the
ones already there: a statement of what the commit does, in plain words. Some
repositories use Conventional Commits prefixes and some do not; either is fine
inside a repository that already mixes them.

Do not add `Co-authored-by:` trailers for tools. Using an AI assistant is fine
and needs no disclosure in the commit, but a trailer naming a bot puts that bot
in the repository's contributor list, where it does not belong.

## The pull request

- Target `main`.
- The title is what a reader of the release notes gets to see.
- The body says what changed, why, and how you know it works.
- CI has to be green. On a first-time contributor's pull request the workflows
  wait for a maintainer to press "Approve and run". That press is not a review,
  it is GitHub asking whether it may execute your branch.
- Draft pull requests are welcome and are a fine way to ask "is this the right
  shape?" before finishing.
- Force-push as much as you like while it is under review. Nothing is merged
  from a stale commit.

Reviews come from two directions. An automated reviewer comments within minutes;
treat it as advisory, because it is often right about details and sometimes
confidently wrong about intent. A human reply can take a few days. Nothing is
merged on the bot's approval and nothing is rejected on its objection.

Pull requests are squash-merged, so your branch becomes one commit and the pull
request title becomes its subject.

## Running the checks yourself

Each repository documents its commands under **Development** in its README, and
CI runs those same commands — read `.github/workflows/ci.yml` if you want the
authoritative list. See [CI.md](CI.md) for what the workflows are expected to
have in common.

There is deliberately no pre-commit framework in these projects. The check that
decides whether a change lands is the one CI runs, and a second copy of that
list in a hook config is one more thing that drifts out of step with it.

## Licence

By opening a pull request you agree that your contribution ships under the
licence of the repository you are contributing to. That is MIT for most of them.
`berndeutsch-for-ai` is split: the code is MIT and the material under `rules/`
is CC BY-SA 4.0. The `LICENSE` file next to the code you touched is the one that
applies.

## Security

Do not open a public issue for a vulnerability. See [SECURITY.md](SECURITY.md).
