# Release topologies

`release.topology` in `.claude/lanes.yml` says how code reaches production. The roles, the work branch and the queue
branch are the same in all five. What changes is the last hop, and so the Release role's gates and commands.

In every topology, **`main`'s commit order is the ship order**. A release takes everything on `main` up to its point.
That is why each milestone arrives on `main` as one promotion from its queue branch.

## `diverged-branch`

```
claude/<work> --PR--> queue/vX.Y.Z --promote--> main --cutover merge--> release/prod
                                                (lower envs)              (prod)
```

- Prod deploys from a release branch that **main is merged into**. Each cutover is a two-parent merge commit, so the
  release branch carries commits `main` does not. A fast-forward push fails.
- **Hazard:** the release branch can accumulate lines `main` does not have (old conflict resolutions).
- **Release adds:** rehearse every cutover on a throwaway branch; diff the merged tree against `main`; require
  byte-identity or a written explanation per surviving file.
- **Pick this when:** production deploys from a long-lived branch and you want a merge commit per cutover as the
  record.

## `ff-branch`

```
claude/<work> --PR--> queue/vX.Y.Z --promote--> main --fast-forward--> release/prod
```

- Prod deploys from a release branch that is fast-forwarded to a commit on `main`. It never diverges.
- It can ship a **prefix** of `main` (stop at an earlier commit), never a subset.
- **Release adds:** before each advance, prove the release branch is an ancestor of the target and the target is on
  `main` (`git merge-base --is-ancestor`).
- **Pick this when:** you want a release branch with no merge-history hazard.

## `tag`

```
claude/<work> --PR--> queue/vX.Y.Z --promote--> main --tag vX.Y.Z--> prod
```

- A version tag on `main` triggers the prod deploy.
- **Release adds:** tag only a commit that is on `main`; the tag message names the milestone.
- **Pick this when:** your deploy pipeline already keys on tags.

## `dispatch`

```
claude/<work> --PR--> queue/vX.Y.Z --promote--> main --dispatch--> prod
```

- A person or agent runs the prod workflow by hand (`workflow_dispatch`), and it deploys `main`'s tip at that
  moment. A push to `main` deploys only the lower environments.
- **Hazard:** a dispatch names a branch, not a commit. A promotion that lands between the lower-environment
  verification and the dispatch ships unverified.
- **Release adds:** confirm `origin/main` equals the verified SHA right before the dispatch, land no promotion in
  between, and confirm the run's `headSha` afterwards.
- **Pick this when:** the prod deploy is a manual button on `main` (store releases, approvals in a GitHub
  environment).

## `main`

```
claude/<work> --PR--> queue/vX.Y.Z --promote--> main (prod)
```

- Every push to `main` deploys prod. **The promotion is the release.**
- **Release adds:** every prod gate runs **before** the promotion, because no later gate exists. Lower-environment
  verification happens on previews or on the queue branch build.
- **Pick this when:** continuous deployment from `main`.

## Protected `main`

Independent of the topology: when `main` requires a pull request (required reviews or status checks), set
`release.promote: pr`. Release then promotes through a PR from the queue branch, merged with `gh pr merge --merge`
(plus `--admin` for a single maintainer). The promotion is still one merge commit.

## Mixed lanes

A project can have dispatch-only prod lanes for most services and a few lanes that deploy prod on any push to `main`
(an edge proxy, a static site). List those paths in `release.main_push_prod_paths`. The roles then treat a promotion
that touches them as a prod step.
