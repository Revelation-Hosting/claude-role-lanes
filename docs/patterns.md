# Patterns that pair with the lanes

These patterns are not rendered into a project by setup. Each one solved a recurring failure on the project the lanes
came from. Adopt any of them through `hard_rules`, `conventions`, `verification.traps` or a role's `extra` text.

## Only-shrinks allow-list guard

**Problem.** An invariant (for example "only the payment service calls the payment provider") can be broken by any
new call site. Review catches some of them.

**Pattern.** A CI test scans the code for calls to the guarded API and fails on any caller outside the choke point.
Existing violations go on an allow-list, each with its issue number. The list **only shrinks**: converting a site
removes its entry in the same PR, and a new money path joins the choke point, never the list.

**In lanes:** a `hard_rules` entry whose `enforcement` names the test.

## Minimum test-count floor

**Problem.** Many test runners exit 0 when they discover zero tests. A misconfigured project then reads as "all
passed".

**Pattern.** Each suite in CI asserts a minimum count of executed tests. The floor rises when tests are added, and
never falls without a written reason.

**In lanes:** `ci.count_floor: true`.

## Bare control query

**Problem.** A query against the wrong log resource, or a retired one, answers zero rows with no error.

**Pattern.** Before trusting a zero-row result, run a count with no filter against the same resource and time range.
A live resource answers with a large number.

**In lanes:** `observability.control`.

## Intent map

**Problem.** The rules for a complex domain (billing, permissions) live only in code. Milestone text describing them
drifts from the code, and sessions plan from the text.

**Pattern.** One file states every decision in the domain as numbered rules, one flow per action, each with a change
log entry that names an issue. Discovery draws the rules from the code on `main` and may edit the file directly.
A billing milestone cites the rule IDs it changes and the rules it must not break. No code reads the map: it states
intent and catches nothing by itself.

**In lanes:** `roles.discovery.may_edit` for the map file, plus `extra` text for Discovery and Development.

## Rehearsed cutover

**Problem.** A release branch advanced by merges can carry lines `main` does not have.

**Pattern.** Merge `main` onto a throwaway branch first. Diff the merged tree against `main`. Require byte-identity,
or a written explanation naming every surviving file.

**In lanes:** built into topology `diverged-branch`.

## Production-shaped fixtures

**Problem.** A canary passes a gate that is broken for real customers, because the test fixture lacks the property
real customers have (an empty optional field, a legacy plan, a second region).

**Pattern.** Pick verification fixtures by matching production data shapes. Name the property that makes the fixture
representative in the runbook gate.

**In lanes:** `verification.traps`.

## Rehearsed migration

**Problem.** A migration passes review and a parse check, then fails on a real database: a constraint, a filtered
index, a session setting.

**Pattern.** Execute the whole migration against a lower environment inside a transaction that rolls back. Run it
twice in one session to prove it is idempotent.

**In lanes:** a `database.rules` line.
