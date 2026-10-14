# Changelog

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
