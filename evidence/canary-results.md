# Repo Sentinel Authority Canary Results

## Scope

This repository is disposable and isolated from
`stacknil/sec-writeups-public`. No scanner, acquisition code, snapshot reader,
materializer, production workflow, or production branch rule was deployed.

All timestamps below are UTC. Repository content was never executed locally.

| Field | Value |
| --- | --- |
| Repository | `stacknil/repo-sentinel-authority-canary` |
| Repository ID | `1368047925` |
| Canonical context | `Repo Sentinel / authoritative gate` |
| GitHub Actions App ID | `15368` |
| Dedicated canary App ID | `4927837` |
| Dedicated App installation ID | `161305593` |

## Dedicated App Permissions

The installation-token response reported:

```json
{
  "repository_selection": "selected",
  "permissions": {
    "metadata": "read",
    "pull_requests": "read",
    "statuses": "write"
  }
}
```

The selected repository was only
`stacknil/repo-sentinel-authority-canary`. The App had no Checks, Contents,
Actions, Workflows, Issues, Administration, or Secrets write permission.

The private key and installation tokens are not evidence artifacts and are not
stored in Git.

## Rule Configurations

The native-source phase used:

```json
{
  "strict": false,
  "checks": [
    {
      "context": "Repo Sentinel / authoritative gate",
      "app_id": 15368
    }
  ]
}
```

The dedicated-App phase used, and the final API readback preserves:

```json
{
  "strict": false,
  "checks": [
    {
      "context": "Repo Sentinel / authoritative gate",
      "app_id": 4927837
    }
  ]
}
```

See [`final-rule.json`](final-rule.json) for the compact final readback.
Immutable run, check, status, source, SHA, and UTC timestamp fields are recorded
in [`timestamped-api-evidence.json`](timestamped-api-evidence.json).

## Automatic GitHub Actions Spoof Evidence

### Modified existing workflow

| Field | Value |
| --- | --- |
| PR | `#1` |
| H | `73db2a6abe34b5b8d146825db596b479d91c748c` |
| B | `f8b1c0b9adf6b240033c3fd3130fd5af5fa8207e` |
| Test merge | `3935cd4cd790a5970a3c4420fd99fdfbc4f4b95c` |
| Run ID | `34740070082` |
| Job/check ID | `103678171058` |
| Event | `pull_request` |
| Check App ID | `15368` (`github-actions`) |
| Result | `success` |

### New standalone workflow

| Field | Value |
| --- | --- |
| PR | `#2` |
| H | `c9030f823a24a5ed998a2ce66c1955f466f435c1` |
| B | `f8b1c0b9adf6b240033c3fd3130fd5af5fa8207e` |
| Test merge | `cc37401bd8f2e4cec2015f2d2f8cadfada829642` |
| Baseline run/job | `34740203019` / `103678517603` |
| Spoof run/job | `34740203090` / `103678517601` |
| Event | `pull_request` |
| Check App ID | `15368` (`github-actions`) |
| Result | `success` |

Both PR-controlled variants created the exact canonical check name under the
ordinary GitHub Actions App identity.

### Fork case

Not executed. The authenticated test environment had one GitHub owner and no
organization, so it could not create a second-owner repository in the same fork
network. An unrelated repository was not substituted because it would not be a
valid fork control.

## Native GitHub Actions Source Matrix

The rule expected App ID `15368`, while the PR-controlled automatic check on H
was successful.

| Trusted Commit Status | Status ID | Automatic check | Merge state |
| --- | --- | --- | --- |
| missing | none | success | `CLEAN` |
| pending | `54063960070` | success | `BLOCKED` |
| failure | `54063973336` | success | `BLOCKED` |
| success | `54063996060` | success | `CLEAN` |

The missing row is decisive: selecting GitHub Actions as the expected source
did not distinguish a trusted workflow from PR-controlled workflow content.

## Dedicated App Source Matrix

The rule expected App ID `4927837`.

| Dedicated App status | Status ID | Actions check | Merge state |
| --- | --- | --- | --- |
| missing | none | success | `BLOCKED` |
| pending | `54065569013` | success | `BLOCKED` |
| failure | `54065593467` | success | `BLOCKED` |
| success | `54065598101` | success | `CLEAN` |
| success | `54065617821` | absent | `CLEAN` |

The missing negative control used PR `#1` at
`93a893153b07df3eac788baea3c763007c07e0e3`. Its automatic canonical check was
successful (`103683402180`, run `34742095420`), but no dedicated-App status was
present and the PR remained blocked.

The App-only positive control used PR `#4` at
`cb029710cc02536cdf50c7f25c858b6ff274bf92`. It had no canonical Actions check,
was blocked before App publication, and became clean after App status
`54065617821`.

## Status and Check Coexistence

Commit Status and Check Run behavior was observed through actual PR merge state,
not inferred from the combined commit-status endpoint.

### Native expected source

- Status success `54064455930`, then Actions failure run/check
  `34741616805` / `103682160107`: `BLOCKED`.
- Status failure `54064481583`, then Actions success run/check
  `34741675629` / `103682308929`: `BLOCKED`.
- Status pending `54064661888`, then Actions success run/check
  `34742095420` / `103683402180`: `BLOCKED`.
- Status success `54063996060`, then status failure `54064046255`:
  `BLOCKED`.
- Status failure `54064046255`, then status success `54064141145`: `CLEAN`.

### Dedicated expected source

- App failure `54065727684`, then Actions success run/check
  `34744808587` / `103690646237`: `BLOCKED`.
- App success `54065755022`, then Actions failure run/check
  `34744858453` / `103690790043`: `BLOCKED`.
- Later App success `54065772072` after the Actions failure: `CLEAN`.

The dedicated source prevents a different source's success from satisfying the
rule. A later same-name Actions failure can still veto temporarily, so expected
source binding is not a denial-of-service boundary. A later dedicated-App
success restored eligibility in the observed sequence.

## Exact Head Movement

PR `#4` moved from:

```text
H1 cb029710cc02536cdf50c7f25c858b6ff274bf92
H2 2487dccbd98848c7872b3052ccc6f889e45012c7
```

After the PR moved to H2, a late H1 success (`54065655693`) left the PR
`BLOCKED`. Publishing H2 success (`54065661260`) changed it to `CLEAN`.

Required property observed:

```text
H1 success did not authorize H2.
```

## Base Movement

The PR head remained
`2487dccbd98848c7872b3052ccc6f889e45012c7` while `main` advanced from:

```text
B1 dc6fbbb774aa680a612cce48179101ca6fa194a5
B2 a83d8f8fedd2155e811babe417be34e0cf8b87bb
```

| Rule mode | Merge state after B2 |
| --- | --- |
| `strict=false` | `CLEAN` |
| `strict=true` | `BEHIND` |
| restored `strict=false` | `CLEAN` |

This is only the observed behavior for this branch-protection configuration.

## Test Merge SHA

For PR `#4` at H2:

| Field | Value |
| --- | --- |
| Head SHA | `2487dccbd98848c7872b3052ccc6f889e45012c7` |
| Test merge SHA | `40cbb567c6d738da2c9a9edb37ed36d8cebe5231` |
| Status target | H2 |
| Canonical statuses on test merge | `0` |
| Canonical checks on test merge | `0` |
| Merge state after H2 App success | `CLEAN` |

In this tested configuration, the required rule evaluated the exact head status;
it did not require the test-merge SHA.

## Source Identity Evidence

The same visible context was published by distinct principals:

```text
github-actions[bot]                    App ID 15368
stacknil-rs-canary-20260913[bot]       App ID 4927837
```

A dedicated-App status sample is captured in
[`app-identity.json`](app-identity.json).

## Final Spoofability Verdict

`INCONCLUSIVE`

Same-repository evidence proves that App ID `4927837` rejects Actions-only
success and accepts only the dedicated App's positive result. However, the
required fork negative control was not executed. The experiment therefore does
not claim the broader `EXPECTED_APP_BINDING_PROVEN` verdict.

## Architecture Consequence

`RESEARCH_MORE`

The remaining experiment is narrow: repeat the decisive missing-App and
App-success rows on a true second-owner fork PR. No production workflow or
signer architecture should be activated from this incomplete canary alone.
