# Test-Merge Authority Gate 1

## Verdict

```text
TEST_MERGE_AUTHORITY_BROKEN
```

The current GitHub test-merge commit cannot be the sole canonical authority
slot for this repository. A required App success on the current test-merge SHA
made the pull request appear merge-eligible, but an actual merge request caused
GitHub to regenerate that SHA before evaluating branch protection. The merge
then failed because the replacement commit had no authoritative status.

The exact evidence is in
[`test-merge-authority-gate-1.json`](test-merge-authority-gate-1.json).

## Scope

This was a canary-only experiment in
`stacknil/repo-sentinel-authority-canary` (repository ID `1368047925`). It used
the disposable App `stacknil-rs-merge-canary-20260915` (App ID `4943620`,
installation ID `161698223`) with only:

- Metadata: read
- Pull requests: read
- Commit statuses: write

The active rule required `Repo Sentinel / authoritative gate` from App ID
`4943620` with `strict: true`.

No production repository was modified, no production App was created, and no
production scanner or repository content was executed locally. Only the
existing harmless canary workflows ran on GitHub-hosted workers.

## Gate 1A

PR #6 bound:

```text
B = 7262fcca3e93385d15e19b850a615c5bc4d2c77a
H = ddd7ed65c62ef2793cb4d2df429117ebb9fe5589
M = 28a83d20190a5cb7b041c4f3d0ea183ee8e09350
```

With no canonical status on `H`, App success status `54164255469` on `M` made
the authenticated pull-request UI report all checks passed and enabled the
merge button. This proved that GitHub can use a status on the test-merge commit
for required-check eligibility.

## Gate 1B

PR #9 kept one `B/H/M` tuple while the dedicated App published controls:

| H | M | Actual eligibility |
| --- | --- | --- |
| success | absent | clean / merge enabled |
| success | pending | blocked |
| success | failure | blocked |
| success | success | clean / merge enabled |

Once the canonical context exists on `M`, its pending or failure result takes
precedence over a success on `H`. However, an absent context on `M` falls back
to the same App/context result on `H`; GitHub does not expose an exclusive
M-only authority namespace.

## Decisive Failure

PR #8 was intended only to advance `main` for the base-movement test. The
failure repeated across three test-merge SHAs:

```text
8400be1476c514abe4ad08e29c2c952657b91f84
  -- App success, merge rejected -->
7de9e1961b3e4e19c16fad2083086c5fa897cb0b
  -- App success, stable polling, merge rejected -->
426e2b40cf3d07f74c1c10a40e03edb72d29fe27
  -- App success, current immediately before request, merge rejected -->
103dcca1fd18610771df6d3e4cf506f4e9b38433
```

The final controlled attempt held both base and head constant:

```text
B = 7262fcca3e93385d15e19b850a615c5bc4d2c77a
H = a596a1ef088a468841962c010e6e6583177453c9
M before = 426e2b40cf3d07f74c1c10a40e03edb72d29fe27
M after  = 103dcca1fd18610771df6d3e4cf506f4e9b38433
```

Status `54165082580` was a dedicated-App success on `M before`. Immediately
before the REST merge request, GitHub still returned that exact SHA as the
current `merge_commit_sha`. The merge endpoint returned HTTP 405 with:

```text
Required status check "Repo Sentinel / authoritative gate" is expected.
```

After the rejected request, `B` and `H` were unchanged, the PR was still open,
and GitHub returned `M after` with a blocked merge state.

## Decision

The merge-commit signer candidate does not advance to v1 adjudication. An
immutable one-shot status protocol can prevent opposite verdict writes, but it
cannot make an unstable GitHub-generated target commit authoritative at the
moment of merge.

Gates 1C, 1D, 1F, and 1G were stopped after this decisive failure. Continuing
them would not restore the failed publication-target invariant. Existing Gate
0 App-source-binding evidence remains valid and independent.
