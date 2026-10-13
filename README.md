# claude-role-lanes

Session roles and a release flow for projects where several Claude Code sessions work at once.

A guided setup interviews you about your project, then writes three agent definitions (**Discovery**,
**Development**, **Release**), a block of hard rules in `CLAUDE.md`, a release-process document, a runbook template
and a bug-report template. Everything it writes is committed to your repository, so every teammate and every session
gets the same rules with no plugin installed.

## The problem it solves

Parallel sessions each make locally correct decisions that are globally wrong. Three sessions find three real bugs,
and each puts its bug into the release in flight. A fix is "verified" by a green build of a binary that does not
contain it. A release branch quietly carries lines `main` does not have. [docs/why.md](docs/why.md) has the case
studies.

The lanes answer with **separation of roles** and **one branch flow** in which the bad states cannot be represented.

## The roles

| Role | Owns | Writes product code | Assigns milestones | Ends at |
|---|---|---|---|---|
| **Default** | General work: questions, one-off fixes, exploration | yes | no | - |
| **Discovery** | Triage, issues, milestone scope, runbook context and invariants | **no** | **yes - the only role that does** | a scoped, frozen milestone |
| **Development** | One milestone or hotfix: code, tests, runbook steps | yes | no | its PR merged into the queue branch |
| **Release** | Promotion, verification, prod, the cutover record | **no** | no (closes shipped ones) | prod, verified |

**The three hard rules** (in `CLAUDE.md`, so they bind every session, Default included):

1. **Anyone files issues. Only Discovery assigns milestones**, after reading the whole board.
2. **File forward, never sideways.** A bug outside your milestone gets a new issue with no milestone and a backlog
   label. A missed site of your milestone's *own* defect is in scope: fix it.
3. **Read the board before any milestone decision**: who else is working, what they decided, the milestone's
   IN SCOPE / OUT OF SCOPE / BELONGS ELSEWHERE.

**The lanes:**

```
claude/<work> --PR--> queue/vX.Y.Z --promote--> main --> prod
 (Development)        (Development)  (Release)         (Release)
```

`main`'s commit order is the ship order, so each milestone arrives on `main` as one promotion. The last hop depends on
your [release topology](docs/topologies.md): a diverged release branch, a fast-forwarded release branch, a tag, a
manual dispatch of `main`, or `main` itself. A protected `main` is promoted through a pull request.

## Quick start

**With the plugin** (in Claude Code, inside your project):

```
/plugin marketplace add Revelation-Hosting/claude-role-lanes
/plugin install lanes@claude-role-lanes
/lanes:setup
```

**Without the plugin:** see [SETUP.md](SETUP.md).

Setup detects what it can (default branch, workflows, environments, test command, migrations directory, existing
labels), then asks about the rest in about seven short rounds. A **Minimal** pace asks only about the project, the
lanes and the deploy, and leaves the optional modules off.

## What setup writes

| File | Content |
|---|---|
| `.claude/lanes.yml` | Your answers. Every other file is rendered from it. |
| `.claude/agents/discovery.md`, `development.md`, `release.md` | The role definitions. |
| `CLAUDE.md` | A block between `<!-- lanes:begin -->` and `<!-- lanes:end -->`: the first-turn role question, the hard rules, the bug-report format, and your project facts. Text outside the markers is never changed. |
| `docs/RELEASE-PROCESS.md` | The shared process document. Path configurable. |
| `docs/runbooks/TEMPLATE-release-runbook.md` | The release runbook, with each section's owner. |
| `.github/ISSUE_TEMPLATE/bug-report.md` | The 9-field bug report. |
| GitHub labels | The backlog label and one label per risk type. Setup asks before it creates them. |

Each generated file carries a marker line. **Delete the marker to take ownership**: setup then never overwrites that
file. A file you already own (no marker) is never overwritten without a question. The recommended choice writes the
new version to `.claude/lanes-new/<path>.new` for a manual merge; the `.new` extension keeps Claude Code from loading
it as an agent.

## Using the roles

| Where | How |
|---|---|
| Terminal | `claude --agent discovery` (or `development`, `release`). Replaces the system prompt; strongest. |
| Claude Desktop | Start a session in the project. The `CLAUDE.md` block makes it ask which role it carries, then load that file. |
| Any session | `@discovery` runs the role as a subagent for one task. |

Typing "you are a discovery agent" does nothing: it never loads the role file.

## Configuration and upgrades

- Edit `.claude/lanes.yml`, then run `/lanes:setup` again to re-render. Reference:
  [config.md](plugins/lanes/skills/setup/reference/config.md).
- Project-specific rules go into the config, not into the generated files: `hard_rules`, `conventions`,
  `verification.traps`, `process.extra` for process rules that bind every role, and `roles.<role>.extra` for text
  that binds one role. An owner ruling that changes a role goes there too, with its date.
- If your `CLAUDE.md` already documents tests, schema changes, runtime queries, local runs and browser checks, set
  `claude_md.project_facts: false`. The block then points at your sections instead of repeating them.
- `/lanes:setup check` compares the rendered output with the files on disk and reports hand edits. It writes nothing.
- After a plugin update, `/lanes:setup` asks only for keys the new version added.
- Examples: [multi-env-platform](examples/multi-env-platform/lanes.yml) (four environments, diverged release branch,
  SQL migrations) and [small-webapp](examples/small-webapp/lanes.yml) (deploy on every push to `main`).

## Requirements

- Claude Code, with project agents (`.claude/agents/`) and skills.
- git, and a GitHub repository. The roles call the `gh` CLI for issues, milestones, labels and PRs.
- Optional: the Claude Desktop session tools. With them, roles read other sessions' transcripts. Without them, GitHub
  issue comments are the board.

## Status and limits (0.1.0)

- **GitHub only.** Issues and milestones are GitHub's. Other trackers are not supported yet.
- **Claude renders the templates.** There is no separate template engine. Setup's verify step checks for unresolved
  tags, frontmatter shape, marker integrity and unchanged text outside the markers, then reads every file through.
- **No hook enforcement yet.** Rule 1 is enforced by the role text, not by a tool-call hook. A `PreToolUse` hook that
  blocks milestone assignment outside Discovery is designed, not shipped, because it has not been tested in a main
  session on Windows and macOS.
- Learned agent memory (`.claude/agent-memory/`) starts empty in each project.

## Contributing

A rendered sentence that is false for your project and that no config key controls is a template defect. Open an
issue with the sentence, the file it rendered into, and the relevant part of your `.claude/lanes.yml` (remove anything
private first).

## Roadmap

- Milestone-assignment guard hook, after it is tested in a main session on Windows and macOS.
- A deterministic reference renderer with snapshot tests of the examples in CI.
- GitLab, Jira and Linear trackers.

## Origin

Built and run in production at Revelation Hosting, a game-server hosting platform, where several Claude Code sessions
work the same repository at once. [docs/why.md](docs/why.md) gives the failures each rule answers.
[docs/patterns.md](docs/patterns.md) gives patterns that pair with the lanes.

## License

[MIT](LICENSE).
