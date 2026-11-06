# Changelog

## 0.1.3 - unreleased

The setup skill now matches `config.md` and `plugin.json`.

- Interview: the `release.topology` question offers all 5 values (issue #3). It listed 4 and left out `dispatch`,
  which the topology inference could still propose. `AskUserQuestion` takes at most 4 options, so the question now
  has 2 steps: `main`, `tag`, `dispatch` or "A release branch", then `diverged-branch` or `ff-branch`.
- Interview: the Round 4 risk-type paragraph moves out of the Pace option list and into Round 4 (issue #4).
- Labels: `config.md` > "Tracker labels" gives the backlog label, the scope-question label and `S1`..`S4` a color and a
  description. Step 8 reads them and no longer invents them. An existing label keeps its own color and description
  (issue #5).
- CHECK mode reports the plugin version that Step 0.4 reads from `plugin.json`, not the literal "0.4" (issue #7).
- `SKILL.md`, `config.md` and `README.md` count 4 core hard rules, not 3, and the README lists rule 4 (issue #10).
  0.1.2 added rule 4 to the CLAUDE.md block and did not update these 3 counts.

## 0.1.2 - unreleased

- No issue closes before its fix takes effect (issue #2). GitHub closes an issue when a closing keyword reaches the
  default branch: in any commit message, in a merge commit's body (it carries the PR title, or the PR body), or in a
  merged PR's body. 0.1.1 banned the keyword only in the queue PR body. Release wrote `Closes #NNN` in every
  promotion, and on 4 of 5 topologies the promotion is not the prod deploy.
- Development: no closing keyword in any commit message, PR title or PR body. Two scans before the queue merge: the
  branch's commit messages, and the PR title and body. At the queue merge, a `Fixed in <queue branch> by PR #N`
  comment on each issue it fixed.
- Release: FIRST ACTION 1 and Gate 1 now require "fix merged in the queue branch, or deferred by Discovery in
  writing", not "issue closed". The old rule could not pass once the close moved to after the promotion.
- Release: new section "When an issue closes". A STEP 1 keyword scan of `origin/<main>..origin/<queue branch>`. On a
  hit, Release promotes, re-opens each issue closed too early, closes it after its gates and records a deviation.
- Release: the promotion writes `Closes #NNN` only on topology `main` or for a docs/process-only fix. Every other issue
  gets `Refs #NNN`, on both the `pr` and the `push` promote paths, and in every other PR into `main`. "After a prod
  release" closes the remaining issues before the milestone.
- Runbook template: the STEP 1 keyword scan, and a Close field per issue: "Closes at merge" or "Refs, closed by
  Release after gates".
- An existing issue fixed as a missed site now gets a close moment (issue #8). Development spawns Discovery to place
  it in the milestone in flight, so Gate 1, the Close field and "After a prod release" cover it. In 0.1.1 such an
  issue never closed. `discovery.md` gets one line for that placement.
- CLAUDE.md block: hard rule 4, which binds Default and Discovery sessions too. Release process document: "When an
  issue closes".
- One scan regex in every file, in portable POSIX ERE form: `(^|[^A-Za-z])` instead of `\b`. UNVERIFIED: `\b` in BSD
  and macOS grep.

## 0.1.1 - unreleased

- CI module: new CLAUDE.md subsection "Waiting for a CI run: background `gh run watch`". A green run sends a Claude
  Code session no notification, and the desktop app's built-in instructions forbid polling CI. Without a project rule,
  a session that waits for green is never woken. The rule names `gh run watch <run-id> --compact --exit-status` as a
  background Bash command, bans Monitor poll loops, and states the override. Development's merge-on-green line and
  Release's run checks point to it.

## 0.1.0 - unreleased

First public version.

- Setup skill `/lanes:setup`: detect, interview, write `.claude/lanes.yml`, render, verify, labels, hand-off.
  `check` mode reports drift without writing.
- Rendered files: `.claude/agents/{discovery,development,release}.md`, a CLAUDE.md block between markers, the release
  process document, a release runbook template, a bug-report issue template.
- Five release topologies: `diverged-branch`, `ff-branch`, `tag`, `dispatch`, `main`. Promotion by push or by PR
  (`release.promote`) for a protected `main`.
- Migrations applied by a runbook step or by the deploy (`database.applied_by`).
- Check V8: no generated file is gitignored; Step 4 proposes a scoped `.gitignore` change. `ci.queue_prs` covers a CI
  workflow that does not run on PRs into queue branches.
- First live install: a private .NET API + Blazor + MAUI repository (topology `dispatch`, PR promotion,
  deploy-applied EF Core migrations).
- Optional modules: house style (ASD-STR100), CI, database migrations, runtime observability, local run, browser
  verification, Claude Desktop session tools.
- Upgrade mode edits `.claude/lanes.yml` in place and keeps comments. Check mode reports drift and writes nothing.
- Verification: 8 checks (V1-V8). A config value the user keeps after a V7 finding is recorded as `ACCEPTED`.
- GitHub issues and milestones only.
