# Security policy

## Reporting a vulnerability

Use GitHub's private reporting form on the repository in question:
**Security → Report a vulnerability**. It is enabled everywhere and opens a
draft advisory that only you and the maintainer can read.

Do not open a public issue, and do not put a working exploit in a pull request.
If the repository is one where you cannot see the form, report it privately in
[sapn95/.github](https://github.com/sapn95/.github/security/advisories/new) and
name the affected project.

Useful in a report: what an attacker gets, the smallest set of steps that shows
it, the version or commit you tested, and the platform. A proof of concept
helps; a proof of concept against somebody else's account does not, and is not
something to send.

## What to expect

These are spare-time projects maintained by one person. There is no bounty and
no service level agreement. What there is: an answer within about a week, a fix
or an explicit "won't fix, here is why", and credit in the advisory unless you
would rather not have it.

Only the latest release of each project gets fixes. There are no maintenance
branches.

## Where the risk actually is

Several of these projects hold credentials for real accounts:

- **epost-mcp**, **ebill-mcp**, **taxme-mcp** — Swiss postal, billing and tax
  portals. Credentials stay on the user's machine. Anything that leaks them,
  sends them anywhere, writes them to a log, or lets a prompt talk the server
  into acting outside its documented scope is in scope here.
- **beeline**, **linkward** — browser extensions with access to browsing
  context. Permission escalation, exfiltration and cross-container leaks are in
  scope.
- **browser-in-a-box** — runs a browser in a container or a VM. Escapes from
  that box are in scope.

Prompt injection reaching an MCP server counts as a vulnerability, not as a
misuse of the tool, wherever it results in an action the user did not ask for.

## Out of scope

- Vulnerabilities in the upstream portals and services themselves. Report those
  to whoever operates them; these projects are only clients.
- Dependency advisories with no reachable path in this code. Dependabot already
  files those, and they get bumped on their own schedule.
- Findings that require an attacker who already has the user's machine, since
  at that point the credentials are theirs anyway.
