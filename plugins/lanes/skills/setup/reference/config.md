# `.claude/lanes.yml` - configuration reference

Setup writes this file into the target repository. Every rendered file is a function of this file and the
templates. Edit it, then run `/lanes:setup` again to re-render.

- **Req** = setup must have a value before it renders. A blank required value stops the render.
- **Default** = the value setup proposes when detection finds nothing.
- **Detect** = where setup looks before it asks.
- A key whose module is off (`enabled: false`) needs no other value in that module.
- A required list may be `[]`.
- **Setup writes every string value double-quoted**, including colors (`"5319E7"`). Unquoted, YAML reads `5319E7` as
  a number and cuts a value at ` #` as a comment. When you edit by hand, quote any value that contains ` #` or `: `,
  starts with one of `` ` * & ! [ { " ' % @ ``, or could read as a number, a date or a boolean.
- Keys whose condition does not apply (for example `branches.release` under topology `main`) may be omitted. Setup
  never asks for them.
- Setup writes every optional key that applies, with `""` or `[]` when it has no value. A key that is present, even
  empty, is never asked again on upgrade.
- A free-text value is rendered as its own sentence after a colon. Write it as one or more complete sentences.

## Top level

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `lanes_version` | string | yes | the plugin version | The scaffold version that last rendered the files. Setup sets it on every write. New keys are found by comparing the file with this reference, not by this value. |

## `project`

| Key | Type | Req | Default | Detect | Meaning |
|---|---|---|---|---|---|
| `project.name` | string | yes | repo name | `gh repo view --json name`, else the folder name | Used in role intros: "You are the Discovery agent for <name>." |
| `project.owner` | string | yes | `the owner` | - | Who rules on scope disputes and exceptions. Written into rulings: "(<owner>, 2026-10-12)". Use a role title, not a personal name, if the repo is public. |
| `project.summary` | string | no | - | README or CLAUDE.md first paragraph | One line: what the product is. Rendered as "Product: <summary>". |

## `style`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `style.ste` | bool | yes | `true` | Adds the ASD-STR100 / Simplified Technical English house style to the CLAUDE.md block. Binds all agent output. |

## `claude_md`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `claude_md.project_facts` | bool | yes | `true` | `true`: the CLAUDE.md block includes the project-fact sections (conventions, tests and CI, schema changes, runtime queries, local runs, browser checks). `false`: the existing CLAUDE.md already documents them outside the markers, and the block only points there. The role files say "see CLAUDE.md" either way, so the facts must exist in CLAUDE.md, inside or outside the block. |

## `paths`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `paths.process_doc` | path | yes | `docs/RELEASE-PROCESS.md` | Where the shared process document is written. |
| `paths.runbook_dir` | path | yes | `docs/runbooks` | Where release runbooks live. The runbook template is written here. |
| `paths.runbook_file` | string | yes | `VX.Y.Z-release-runbook.md` | The runbook file-name pattern. `VX.Y.Z` stands for the milestone version. |

## `tracker`

GitHub issues and milestones only in this version.

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `tracker.backlog_label` | string | yes | `needs-milestone` | The label on every issue with no milestone. `is:open label:<it>` is the unscheduled backlog. |
| `tracker.scope_question_label` | string | no | `discovery-question` | Marks an open scope question for Discovery. Empty string disables it. |
| `tracker.severity_labels` | bool | yes | `false` | Creates labels `S1`..`S4`. |
| `tracker.bug_report_max_words` | int | yes | `300` | Word ceiling for a bug report body. |
| `tracker.repro_inputs` | string | no | - | A phrase naming the inputs a reproduction must state. Example: `tenant ID shape, plan, region, game title`. |
| `tracker.population_examples` | string | no | - | A phrase naming this project's customer segments. Example: `tenants in one region, tenants of one game title`. |
| `tracker.risk_types` | list | yes | the 6 core types | One GitHub label per type, named exactly `name`. Each item: `name`, `color` (6 hex digits, no `#`), `definition` (one or two sentences; what the report must name). |

**Core risk types** (setup proposes all 6):

| name | color | definition |
|---|---|---|
| `LEGAL/CONTRACTUAL` | `B60205` | We promise X in a contract, SLA, ToS or marketing and do not deliver it. Name the promise and cite where it is written. |
| `FINANCIAL` | `D93F0B` | Money moves wrong: overcharge, undercharge, missed charge, double charge, tax error. State what "should" is (the correct amount and its source). |
| `SECURITY/EXPLOIT` | `5319E7` | Unauthenticated endpoint, IDOR, injection, secret exposure, privilege escalation. Name the missing control and the worst action an attacker can take with it. |
| `RACE/STABILITY` | `0E8A16` | Concurrency bug, deadlock, crash, corruption under load. State the interleaving that triggers it. |
| `CUSTOMER HARM` | `E99695` | A customer's service, data, uptime or access is harmed. Name the customer asset at risk. |
| `COSMETIC/UX` | `C5DEF5` | None of the above. Add one sentence on why it still matters. |

**Extended risk types** (setup offers these as a multi-select):

| name | color | definition |
|---|---|---|
| `DATA-INTEGRITY` | `B60205` | A stored row breaks an assumption a check depends on, so the check silently skips it. |
| `OBSERVABILITY/FORENSICS` | `0E8A16` | A customer or incident question cannot be answered because the evidence was never recorded. |
| `MONITORING-BLINDNESS` | `FBCA04` | A monitor reports healthy while the monitored thing is missing, dead or unmonitored. |
| `ENV-DRIFT` | `5319E7` | An environment is correct today and wrong after a rebuild, refresh or upgrade. |
| `OPERATOR-TOOLING` | `5319E7` | An internal operator or admin tool fails on a legitimate input. No customer, no money, no data loss. |
| `DELIVERY-PIPELINE` | `1D76DB` | CI/CD speed or cost: work waits on, or pays for, the pipeline. No customer, money or data. |
| `PERFORMANCE` | `1D76DB` | Speed or efficiency with customer-visible impact. Not a defect. |
| `GROWTH/ACQUISITION` | `006B75` | A defect in the acquisition funnel loses prospects before they become customers. |

## `roles`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `roles.model` | string | yes | `opus` | Agent frontmatter `model`. `inherit` uses the session model. |
| `roles.effort` | string | yes | `high` | Agent frontmatter `effort`. |
| `roles.discovery.may_edit` | list | no | `[]` | Files Discovery may edit as an exception to "writes no product code". Each item: `path`, `why`. |
| `roles.discovery.extra` | markdown | no | - | Appended to the Discovery role under "Project rules for this role". |
| `roles.development.extra` | markdown | no | - | Same, for Development. |
| `roles.release.extra` | markdown | no | - | Same, for Release. |

## `coordination`

| Key | Type | Req | Default | Detect | Meaning |
|---|---|---|---|---|---|
| `coordination.desktop_session_tools` | bool | yes | `false` | ask; the setup session's own tools say nothing about other contributors | `true`: most sessions run in Claude Desktop, so the roles read the board with its session tools (list, search transcripts, send message). A terminal session falls back to GitHub. `false`: GitHub is the board, and sessions leave `WORKING:` / `DONE:` comments on the umbrella issue. |

## `branches`

| Key | Type | Req | Default | Detect | Meaning |
|---|---|---|---|---|---|
| `branches.main` | string | yes | `main` | `gh repo view --json defaultBranchRef` | The integration branch. |
| `branches.work` | string | yes | `claude/<work>` | - | Work-branch pattern. |
| `branches.queue` | string | yes | `queue/vX.Y.Z` | - | Queue-branch pattern. One per milestone. |
| `branches.release` | string | topology `diverged-branch` or `ff-branch` | `release/prod` | `git branch -r` | The branch prod deploys from. |

## `release`

| Key | Type | Req | Default | Detect | Meaning |
|---|---|---|---|---|---|
| `release.topology` | enum | yes | - | workflow triggers | `diverged-branch`, `ff-branch`, `tag`, `dispatch` or `main`. See `docs/topologies.md` in the claude-role-lanes repo. |
| `release.dispatch_command` | string | topology `dispatch` | - | the prod workflow's `workflow_dispatch` inputs | The exact command that dispatches the prod deploy. Example: `gh workflow run ci.yml --ref main -f environment=prod`. |
| `release.promote` | enum | yes | `push` | branch protection on `branches.main` | `push`: Release pushes the promotion merge commit to `branches.main`. `pr`: `branches.main` is protected, so Release opens a PR from the queue branch and merges it. |
| `release.pr_merge_args` | string | `release.promote` is `pr` | `--merge` | protection rules | Arguments for `gh pr merge`. Must contain `--merge` (a merge commit, never a squash). Add `--admin` when a single maintainer bypasses a required review. |
| `release.tag` | string | topology `tag` | `vX.Y.Z` | `git tag` | Tag pattern that deploys prod. |
| `release.prod_deploy` | string | yes | - | prod workflow triggers | One or two sentences: how prod is deployed. Include the exact command if a person or agent runs one. |
| `release.record_name` | string | yes | `Production Cutover Record` for `diverged-branch`, else `Production Release Record` | - | The name of the runbook section Release appends after prod. Keep an existing name if the project's runbooks already use one. |
| `release.prod_lanes` | list | no | `[]` | `.github/workflows/*` | Each item: `name`, `workflow` (the file name, for example `web-cd-prod.yml`), `trigger`, and optional `paths` (the paths whose change fires this lane). Rendered as the canonical prod-lane table. |
| `release.main_push_prod_paths` | list | no | `[]` | push-triggered prod workflows | Paths whose merge to `main` deploys prod directly. Each such promotion IS a prod step. |

**Topology summary:**

| Value | Prod deploys from | Advanced by |
|---|---|---|
| `diverged-branch` | `branches.release` | A two-parent merge of `main` into it. A fast-forward push fails. |
| `ff-branch` | `branches.release` | A fast-forward to a commit on `main`. |
| `tag` | A tag on `main` matching `release.tag` | Tagging a commit on `main`. |
| `dispatch` | `main`'s tip at the moment of a manual dispatch | Running `release.dispatch_command`. |
| `main` | `main` | Every push to `main`. The queue -> main promotion is the prod deploy. |

## `process`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `process.extra` | markdown | no | - | Appended to the process document under "Project-specific process". For process rules that bind every role: freeze periods, deploy variables, configuration-change procedures. |

## `environments`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `environments.prod` | string | yes | `prod` | Name of the production environment. |
| `environments.lower` | list | no | `[]` | Lower environments, in promotion order. Example: `[dev, test, stage]`. |
| `environments.lower_deploy` | string | when `lower` is not empty | - | How the lower environments receive code. Example: "CD on push to main, path-filtered, dev then test then stage in one run." |

## `ci`

| Key | Type | Req | Default | Detect | Meaning |
|---|---|---|---|---|---|
| `ci.enabled` | bool | yes | - | `.github/workflows/*` with `pull_request` | A test workflow exists. |
| `ci.workflow` | path | when enabled | - | same | The test workflow file. |
| `ci.command` | string | when enabled | - | package scripts, `*.sln`, `Makefile`, the workflow's test step | The command a person types to run the tests locally (`npm test`, not the script body). It renders inside a code span, so it must be a runnable command, not prose. |
| `ci.queue_prs` | bool | when enabled | - | the workflow's `pull_request:` filter | `true`: CI runs on a PR into a queue branch (no `branches:` filter, or a filter that matches the queue pattern). `false`: the roles tell Development to run `ci.command` locally on the PR head and paste the result. |
| `ci.blocks_merge` | bool | when enabled | `false` | branch protection | `false`: a red check is information, not a gate. |
| `ci.count_floor` | bool | when enabled | `false` | - | `true`: each suite asserts a minimum test count, because a runner that discovers 0 tests can exit 0. |

## `database`

| Key | Type | Req | Default | Detect | Meaning |
|---|---|---|---|---|---|
| `database.enabled` | bool | yes | - | `migrations/` dirs, ORM config | The project has a schema that changes. |
| `database.engine` | string | when enabled | - | connection strings, packages | Example: `PostgreSQL`. |
| `database.migrations_dir` | path | when enabled | - | a `migrations/` or `Migrations/` directory, `db/migrate/`, a Prisma or Alembic migrations folder | Where every migration script goes. |
| `database.base_schema_dir` | path | no | - | - | Base-schema definitions kept true alongside migrations. |
| `database.applied_by` | enum | when enabled | `runbook` | startup code calling a migrate API (`Database.Migrate`, `MigrateAsync`, `migrate deploy` in an entrypoint) | `runbook`: a runbook step applies each migration per environment. `deploy`: the application or its deploy applies pending migrations; the runbook lists and verifies them. |
| `database.apply` | string | when enabled | - | - | How a migration is applied: the command and who runs it. Not where in the runbook (the templates say that). Example: "`npm run migrate`, run by Release, once per environment." |
| `database.rules` | list | no | `[]` | - | Migration-script rules. Each is one line. Example: "Every migration is idempotent." |

## `observability`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `observability.enabled` | bool | yes | - | Agents can query runtime logs or traces from this workstation. |
| `observability.tool` | string | when enabled | - | Example: `Grafana Loki via logcli`. |
| `observability.query` | string | when enabled | - | The command an agent runs. Use `<QUERY>` for the query text. If it needs a resource ID, put the IDs or the command that lists them in `observability.notes`. |
| `observability.control` | string | no | - | A bare control query that must return a large non-zero count. An agent runs it before it trusts a zero-row result. |
| `observability.notes` | list | no | `[]` | One line each. Example: "Resources named `old-*` are retired and answer every query with 0 rows." |

## `local_run`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `local_run.enabled` | bool | yes | - | Some component runs on a workstation. |
| `local_run.command` | string | when enabled | - | How to start it. |
| `local_run.allowed_envs` | list | when enabled | `[]` | Environments a local run may point at. Prod should not be on this list. |
| `local_run.not_local` | list | no | `[]` | Components that cannot run locally. One line each, with how to verify them instead. |

## `browser`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `browser.enabled` | bool | yes | - | Agents drive a browser to verify UI. |
| `browser.tool` | string | when enabled | - | Example: `Playwright MCP (mcp__playwright__*)`. |
| `browser.sign_in` | string | no | - | How an agent gets an authenticated session without typing credentials. Agents never type credentials. |

## `conventions`, `hard_rules`, `verification`

| Key | Type | Req | Default | Meaning |
|---|---|---|---|---|
| `conventions` | list of strings | no | `[]` | The project conventions that cause the most rework when missed. One line each. Rendered into CLAUDE.md and the Development role. |
| `hard_rules` | list | no | `[]` | Project hard rules, added after the 3 core hard rules. Each item: `title`, `rule`, `enforcement` (what fails when it is broken; may be empty), `why` (the measured cost that produced the rule). |
| `verification.traps` | list of strings | no | `[]` | Project-specific ways a check passes while the system is broken. One line each. Rendered into the verification standard. |
| `verification.accepted` | list | no | `[]` | Config values the user confirmed after setup's read-through (V7) flagged them. Each item: `key`, `reason`. V7 does not flag a listed key again while its value is unchanged. |
