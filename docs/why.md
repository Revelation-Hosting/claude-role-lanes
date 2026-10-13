# Why the lanes exist - case studies

These lanes were not designed up front. Each rule replaced a failure that happened, on a production platform, with
several Claude Code sessions working in parallel. The cases below are generalized: version numbers, issue numbers,
customer counts and vendor names are removed. The mechanism of each failure is kept, because the mechanism is what
the rule answers.

## 1. A global decision made from local information

**What happened.** A one-line request asked for a billing retry window. Three sessions worked in parallel on related
findings. Each found something real. Each reasoned, correctly from what it could see, that its find belonged in the
release in flight. The release grew to span four unrelated subsystems: billing, an operations dashboard, admin
configuration pages and a schema refactor.

**Why it happened.** Nobody was careless. Choosing a release for an issue is a *global* decision: it depends on
everything else in flight. Each session made it from *local* information.

**The rule.** Anyone files issues. Only Discovery assigns milestones, and only after it reads the whole board. A
homeless issue carries a backlog label until Discovery places it.

## 2. "Verified" against the wrong binary

**What happened.** A fix was reported verified on the strength of a green build and an argument that the code path
was correct. The test ran against a build that did not contain the fix: the change had been reverted out of the
working tree that was running. Only clicking the real control caught it.

**The rule.** A green build is not verification. Neither is a code-path argument. Verify the artifact that ships:
fetch what the client actually receives, and confirm the running process is built from the branch you think.

## 3. An audit finding about code that never runs

**What happened.** An audit filed a confident finding against a view template. No code path rendered that view. A
Development session that trusted the issue would have spent hours fixing dead code.

**The rule.** Discovery confirms a finding is reachable before filing it. Development is expected to refute the brief:
it opens every citation and corrects an issue that is wrong.

## 4. The wrong install measured

**What happened.** A session read the version of the command-line tool on `PATH` to decide which features were
available. The desktop app ran its own, newer runtime. The old number produced a chain of wrong conclusions about what
the platform could do.

**The rule.** When a check comes back clean, confirm you checked the thing that executes. A search of the wrong path,
a version from the wrong install and a query against a retired resource all read exactly like a real absence.

## 5. A query that answered "nothing happened"

**What happened.** An investigation asked whether a set of web-form submissions had been recorded. It queried two
retired telemetry resources. They answered every query with zero rows and no error. The investigation concluded the
data was lost. The live resource held every submission.

**The rule.** Run a bare control query (a count with no filter) before trusting a zero-row result. Zero on the control
means the wrong resource, not a real absence.

## 6. A release branch that carried what main did not

**What happened.** The production branch was advanced by merging `main` into it. Over many cutovers it accumulated
lines that `main` did not have: stale versions from old conflict resolutions. The model in everyone's head was "the
release branch points into main", so nothing gave a reason to look.

**The rule.** Rehearse every cutover: merge `main` onto a throwaway branch and diff the merged tree against `main`.
Require byte-identity, or a written explanation that names every surviving file.

## 7. Filing forward what the release was built to fix

**What happened.** The rule "file forward, never sideways" worked so well that it overshot. In one batch of
releases, most issues filed forward were the release's *own* defect at a site the scope list had missed. Issues filed
outpaced issues fixed by about two to one.

**The rule.** A missed site of the milestone's own defect is in scope. Development quotes the milestone line the
finding breaks. A clear match is fixed in place. A close call goes to Discovery. Only a finding that breaks no line of
the contract is filed forward.

## 8. One defect, fixed one site at a time

**What happened.** The same class of defect (a duplicated side effect) was found and fixed site by site, many times,
each time with a different local guard. The guards disagreed with each other.

**The rule.** For an invariant that many call sites can break, build one choke point and a CI guard that fails any
caller outside it. The guard's allow-list names each unconverted site and only shrinks. In lanes, such a rule is a
**project hard rule** (`hard_rules` in `.claude/lanes.yml`). See [patterns.md](patterns.md).
