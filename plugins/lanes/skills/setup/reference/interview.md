# Detection and interview

Setup **detects first and asks second**. Every question shows the detected value, or the default, as its first option.
The user confirms or changes it.

## Detection (read-only)

Run each probe. A probe that fails, or finds nothing, leaves its key undetected; it is never an error. Record the
evidence for each detected value (the command and the line it returned). Show the evidence when you ask.
**Never count `.claude/lanes.yml` itself as evidence.**

| Key | Probe, in order; the first hit wins |
|---|---|
| `project.name` | The manifest name: `package.json` `name`, `pyproject.toml` `[project] name`, `Cargo.toml` `[package] name`, the last segment of the `go.mod` module, the `*.sln` file name. Then the first `# ` heading of `README.md` or `CLAUDE.md`. Then `gh repo view --json name --jq .name`. Then the folder name of `git rev-parse --show-toplevel`. |
| `project.summary` | The first paragraph of `README.md`, else of `CLAUDE.md`, that is not a heading, a badge line or a list. |
| `branches.main` | `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`. Then `git symbolic-ref --short refs/remotes/origin/HEAD`. Then, with no remote, a local branch named `main` or `master` (`git branch --list main master`), then `git branch --show-current`. Never `git config init.defaultBranch`: it is the machine default for new repositories, not this repository's branch. |
| Remote branches | `git branch -r --format="%(refname:short)"`. Look for `release/*`, `prod`, `production`, `stable`, `queue/*`. |
| Tags | `git tag --sort=-creatordate` (first 5 lines). |
| Workflows | List `.github/workflows/*.yml` and `*.yaml`. Read each file's `on:` block, its `environment:` keys, its `run:` steps, and any `ref:` pinned in a checkout step. |
| Environments | `gh api repos/:owner/:repo/environments --jq ".environments[].name"`, plus every `environment:` value in the workflows. |
| `ci.*` | The CI workflow is one triggered on `pull_request` that runs tests. The local command is the command a person types: `npm test` when `package.json` has `scripts.test`; `dotnet test` for a `*.sln` or `*.csproj`; `pytest` for `pyproject.toml`; `go test ./...` for `go.mod`; `cargo test` for `Cargo.toml`; `make test` for a `Makefile` `test:` target. Else the workflow's own test `run:` line. |
| `ci.queue_prs` | The CI workflow's `on: pull_request:` block. No `branches:` filter, or a filter that matches the queue pattern (for example `queue/**`), means `true`. A filter that names only other branches means `false`; offer the one-line fix (add the queue pattern to `pull_request.branches`) and say which jobs would then run on such a PR. |
| `ci.blocks_merge` | `gh api repos/:owner/:repo/branches/<main>/protection`. HTTP 404 means no protection, so `false`. Any other failure leaves it undetected. |
| `database.*` | Any of: a `migrations/` or `Migrations/` directory, `prisma/schema.prisma`, `alembic.ini`, `db/migrate/`, `flyway.conf`, `knexfile.*`, `*.sql` files under a `db/` or `sql/` directory. |
| `observability.*` | A reference in the project's code or config to Application Insights, Datadog, Sentry, OpenTelemetry, Grafana, Loki, Honeycomb or New Relic. A CLI on `PATH` (`az`, `gcloud`, `aws`, `logcli`) is supporting evidence only, never a detection by itself. |
| `browser.*` | `.mcp.json` naming a Playwright server; `@playwright/test`, Cypress or Selenium in the package manifests. |
| `local_run.*` | `package.json` `scripts.dev` or `scripts.start` (propose `npm run dev` / `npm start`); a `launch.json` or `Procfile`; a run command in `README.md` or `CLAUDE.md`. |
| Labels | `gh label list --limit 200 --json name,color,description`. |
| Candidates | The existing `CLAUDE.md`. Sections and lines marked `CRITICAL`, `IMPORTANT`, `NEVER`, `ALWAYS` or `MUST` are candidate `conventions` and `hard_rules`. A `CLAUDE.md` section that documents a module (tests and CI, conventions, schema changes, runtime queries, local runs, browser checks) is a **covered** module. |

`coordination.desktop_session_tools` is **not detected**. That this setup session has the Claude Desktop session tools
says nothing about the other contributors. Ask (Round 7).

### Prod deploy workflow

A workflow (or a job in it) is a **prod deploy workflow** when it deploys and targets production: it has an `environment:` named like
`prod` or `production`, or a job or file name with `prod`, `production`, `deploy`, `release` or `publish` together
with a deploy step (a cloud CLI, `kubectl`, `scp`, `rsync`, a hosting provider action). A workflow that only builds
or tests is not one, whatever its trigger.

### Topology inference

| Evidence | Proposed `release.topology` |
|---|---|
| A prod deploy workflow is triggered by `push: tags:` | `tag` |
| A prod deploy workflow is triggered by `push:` to the main branch | `main` |
| A prod deploy job runs only on `workflow_dispatch` and requires the main branch (`if: github.ref == 'refs/heads/main'`) | `dispatch` |
| A prod deploy workflow checks out or is triggered by another branch (for example `release/prod`), and `git rev-list --count origin/<main>..origin/<that branch>` is above 0 | `diverged-branch` |
| Same, and that count is 0 | `ff-branch` |
| No prod deploy workflow found | No proposal. Ask, with no option marked Recommended. |

A push-triggered prod workflow that exists **beside** dispatch-only prod lanes is a `release.main_push_prod_paths`
candidate. Propose its `paths:` filter.

## Pace

Before Round 1, ask one question:

- **Guided (Recommended)** - all 7 rounds. Most answers are pre-filled confirmations.

In Round 4, also propose risk types that fit the product's domain, not only the catalogue (for example `LIFE-SAFETY`
for an emergency-response product, `LOCATION-PRIVACY` for one that stores where vulnerable people are). Mark them
"(proposed)" and give each a definition in the config.md style.
- **Minimal** - Rounds 1, 2 and 3 only. Every optional module is written as `enabled: false`. The user turns one on
  later by editing `.claude/lanes.yml` and running `/lanes:setup` again.

Use `AskUserQuestion` with at most 4 questions per call and at most 4 options per question. When a round's values were
all detected, ask one question that lists them and offers "Confirm all (Recommended)" and "Change some". Free-text
answers (lists, rules, commands) are asked in chat, one prompt per value, with the detected value quoted for the user
to edit.

Keys marked **not asked** below are written with their default. The user changes them by editing the file. These
keys are never asked: `lanes_version` (the plugin version), `paths.process_doc`, `paths.runbook_dir` and
`paths.runbook_file` (defaults from config.md), `tracker.bug_report_max_words`, `roles.model`, `roles.effort`.

A choice between fixed options uses `AskUserQuestion`. A single free-text value (a name, a sentence, a command) is
asked in chat.

## Round 1 - Project

| Key | Question |
|---|---|
| `project.name` | "Project name used in the role files?" |
| `project.owner` | "Who rules on scope disputes and exceptions? Use a role title if this repo is public." Options: `the owner` (Recommended), `the tech lead`, `the maintainers`. Free text is allowed. |
| `project.summary` | "One sentence: what is the product?" (optional) |
| `style.ste` | "Add the ASD-STR100 / Simplified Technical English house style to CLAUDE.md? It binds all agent output: short sentences, no hedging, numbers over adjectives, `UNVERIFIED:` marks." Options: Yes (Recommended), No. |

## Round 2 - Lanes

| Key | Question |
|---|---|
| `release.topology` | "How does code reach prod?" Options: the four values, with the proposed one first, marked Recommended, and its evidence in the description. Descriptions: `diverged-branch` - "a release branch advanced by merging main into it"; `ff-branch` - "a release branch fast-forwarded to a commit on main"; `tag` - "a version tag on main deploys"; `main` - "every push to main deploys". |
| `branches.main` | Confirm the detected default branch. |
| `branches.queue` | "Queue-branch pattern, one per milestone?" Default `queue/vX.Y.Z`. |
| `branches.work` | "Work-branch pattern?" Default `claude/<work>`. |
| `branches.release` | Topology `diverged-branch` or `ff-branch` only. Default: the detected branch, else `release/prod`. |
| `release.tag` | Topology `tag` only. Default: the pattern of the latest tag, else `vX.Y.Z`. |
| `release.dispatch_command` | Topology `dispatch` only. Propose it from the prod job's `workflow_dispatch` inputs, for example `gh workflow run ci.yml --ref main -f environment=prod`. |
| `release.promote` | Detected from `gh api repos/:owner/:repo/branches/<main>/protection`: required PR reviews or required status checks mean `pr`. Confirm. |
| `release.pr_merge_args` | Only for `pr`. "Required reviews block a single maintainer. Merge with `--admin`, or wait for a second reviewer?" Options: `--merge --admin`, `--merge`. |
| `release.record_name` | Not asked, unless existing runbooks under `paths.runbook_dir` use a name for the post-prod record; then confirm that name. Default by topology (config.md). |

## Round 3 - Environments and deploy

| Key | Question |
|---|---|
| `environments.prod` | Confirm the prod environment name. |
| `environments.lower` | "Lower environments, in promotion order?" Detected list first. "None" is valid. |
| `environments.lower_deploy` | Only when `environments.lower` is not empty. "How do the lower environments receive code? One sentence." Propose one from the workflow triggers. |
| `release.prod_deploy` | "How is prod deployed? One or two sentences, with the exact command if a person or agent runs one." Propose one from the prod deploy workflows, for example "Dispatch each prod workflow with `gh workflow run <file> --ref release/prod`." |
| `release.prod_lanes` | Only when detection found prod deploy workflows. Propose one row per workflow (`name`, `workflow`, `trigger`, `paths` from its `paths:` filter). Ask to confirm, edit or drop the table. Otherwise not asked; `[]`. |
| `release.main_push_prod_paths` | Only if detected. "These workflows deploy prod on a push to main. Should the roles treat a promotion that touches these paths as a prod step?" |

## Round 4 - Tracker and bug reports

| Key | Question |
|---|---|
| `tracker.backlog_label` | "Label for issues with no milestone (the unscheduled backlog)?" Default `needs-milestone`; an existing label of similar meaning first if one is detected. |
| `tracker.scope_question_label` | "Label for an open scope question to Discovery?" Default `discovery-question`. "None" disables it. |
| `tracker.risk_types` | The 6 core types are always included. Offer the 8 extended types (config.md) as two multi-select questions of 4 options each, in one `AskUserQuestion` call. No option is pre-selected. Then, in chat: "Rename `CUSTOMER HARM` to name your own customer asset (for example `HOSTED-SERVICE HARM`)?" Existing labels with a matching meaning keep their existing name and color. |
| `tracker.severity_labels` | "Create labels S1-S4?" Default No. |
| `tracker.repro_inputs`, `tracker.population_examples` | In chat, optional: "Which inputs must every reproduction state (for example: tenant ID shape, plan, region)? Which customer segments do you count by (for example: tenants in one region)?" |
| `tracker.bug_report_max_words` | Not asked. Default 300. |

## Round 5 - Verification

Ask each module's `enabled` first. Ask the rest of a module only when it is on.

| Key | Question |
|---|---|
| `ci.*` | Confirm the detected workflow and local command. "Does a red CI check block merge?" "Does each suite assert a minimum test count?" |
| `local_run.*` | "Can an agent run the app on this workstation?" Detected command first. Then: "Which environments may a local run point at?", only when `environments.lower` is not empty; prod is never offered. Then: "Which components cannot run locally, and how is each verified instead?" (chat, one line each) |
| `browser.*` | "Which browser tool verifies UI?" Detected tool first. "How does an agent get a signed-in session without typing a credential?" (optional) |
| `observability.*` | "Can an agent query runtime logs or traces from here? Which tool? The exact query command, with `<QUERY>` for the query text? Where do the resource IDs come from?" Then: "A bare control query that always returns a large count?" (optional, recommended) |
| `verification.traps` | "Name any way a check here passes while the system is broken. One line each." Example to show: "The collector jobs swallow exceptions, so a run with no error log proves nothing; check the written row." (optional) |

## Round 6 - Data

| Key | Question |
|---|---|
| `database.enabled` | "Does the project have a schema that changes?" Detected evidence in the description. |
| `database.engine`, `migrations_dir`, `base_schema_dir` | Confirm detected values. |
| `database.applied_by` | Detected when startup code calls a migrate API. "Who applies a migration: a runbook step, or the deploy?" |
| `database.apply` | "How is a migration applied: the command, and who runs it?" Example: "`npm run migrate`, run by Release, once per environment." |
| `database.rules` | "Migration-script rules, one line each?" Offer defaults: "Every migration is idempotent.", "A migration never edits a migration that already ran in any environment." (optional) |

## Round 7 - Project rules and roles

| Key | Question |
|---|---|
| `hard_rules` | Ask in chat: "Which rule, when broken, has cost this project the most? Give the rule, what enforces it (a test, a CI check, nothing yet), and what breaking it cost." Repeat for up to 3 rules. Offer the CLAUDE.md candidates found in detection. "None" is valid. |
| `conventions` | Multi-select the CLAUDE.md candidates found in detection (4 per question), then ask in chat for others. One line each. |
| `claude_md.project_facts` | Only when detection found covered modules in an existing CLAUDE.md. "CLAUDE.md already documents <the covered modules>. Leave the project facts out of the lanes block?" Mark "Yes" -> `false` Recommended only when **every** enabled module is covered (and `conventions` is empty or covered). Otherwise mark "No" -> `true` Recommended. Otherwise not asked; default `true`. |
| `process.extra` | "Any process rule that binds every role and has no other slot (a freeze period, deploy variables, a config-change procedure)?" (optional, chat) |
| `roles.discovery.may_edit` | "Are there files Discovery may edit as an exception to writing no product code (for example a decision map it maintains)?" Default none. |
| `roles.<role>.extra` | "Any rule that binds one role only?" Default none. |
| `roles.model`, `roles.effort` | Not asked. Defaults `opus` / `high`. |
| `coordination.desktop_session_tools` | "Where do most sessions on this project run?" Options: "Terminal or IDE (Recommended)" -> `false`; "Claude Desktop" -> `true`. Explain: with `true`, the roles read other sessions through the Desktop session tools and fall back to GitHub in a terminal; with `false`, GitHub comments are the board. |
