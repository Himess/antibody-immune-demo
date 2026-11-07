# Antibody immune memory: a demo

Acme Lend is a fictional lending protocol with planted bugs, a **deliberately vulnerable test fixture** that
demonstrates [Antibody](https://github.com/Himess/antibody): never deploy it to mainnet. Its program on
near-antibody (NEAR testnet, program 5, anchored at Sepolia block 12063269) paid for four proofs; two rounds of
fixes followed, and the final fix shipped through the release gate.

This repository is Acme Lend at that final fix, as it shipped on testnet: `AcmeLendFinal` and
`AcmeVaultFixed` (`src/acme-fixes`) over the contracts they fix (`src/acme`), with the compiler settings it was
deployed with. Its build reproduces the code the release permits installed, byte for byte.

On every pull request, a GitHub Action re-runs each paid proof of program 5 that it can read against the
pull request's build, with Antibody's verifier, locally, at the program's anchor, and fails the check if one
still breaks a committed rule: the pull request would re-open a paid hole (Antibody's `docs/immune.md`).

## What is here

| Path | What |
|---|---|
| `.antibody/immune.toml` | The mapping: program 5, and the program target each contract builds (`AcmeLendFinal` replaces the `AcmeLendImpl` implementation, `AcmeVaultFixed` the `AcmeVaultImpl` one), with their constructor arguments as deployed |
| `.antibody/proofs/` | The sealed blobs of the paid commits 13, 14, 18 and 19, public on chain since their reveal (read with read-only RPC calls) |
| `.antibody/keys/13.hex` | Commit 13's content key, already published in the Antibody repository (`fixtures/zk/a`) |
| `.github/workflows/immune.yml` | The check, on `pull_request` (never `pull_request_target`) |
| `patches/` | Two pull requests, prepared as patches |
| `results/` | The check run locally against each build (view calls and local verifier runs only) |

## The paid proofs

| Commit | Paid for | The bug (public in Antibody's docs) | Readable by the check today |
|---|---|---|---|
| 13 (agent-a) | `solvency` (critical), 0.1 wNEAR | #1: collateral at the pool's spot price | yes, its content key is published |
| 14 (agent-b) | `fees_to_treasury` (high), 0.03 wNEAR | #2, bug B: `sweepFees` without access control | not yet: sealed |
| 18 (agent-f) | `solvency` (critical), 0.1 wNEAR | #3: `repay` ignores `transferFrom`'s result | not yet: sealed |
| 19 (agent-g) | a gap claim, 0.01 wNEAR | #5: `depositFor` takes anyone's approved funds | sealed (a gap claim has no tag to disclose) |

Commits 14 and 18 qualify for disclosure now: they were paid and the program's disclosure delay has passed.
One `request_disclosure` call (permissionless; whoever asks pays for the run) and its run's T1 confirmation
make them readable by anyone, pull requests from forks included. Until then the check lists them as not
re-run. The project key (a repository secret, `ANTIBODY_PROJECT_KEY`) reads a paid finding once its release
wrap exists (`antibody program read --commit <id>`, one transaction each).

## Two pull requests

| Branch | Patch | Change | Expected |
|---|---|---|---|
| `red/unguarded-fee-sweep` | `patches/red-unguarded-fee-sweep.patch` | Drops the admin check from `AcmeVaultFixed.sweepFees`: bug B comes back | **red** (exit 2), naming commit 14, rule `fees_to_treasury` (high), entry point `AcmeVault.sweepFees(address)`, once commit 14 is readable. Until then: green, with commit 14 listed as not re-run (exit 3 with `strict: true`) |
| `green/pending-fees-view` | `patches/green-pending-fees-view.patch` | Adds a view, `AcmeVaultFixed.pendingFees()`: new runtime code, same behaviour | **green** (exit 0) |

## Local results

`results/README.md` has the runs against testnet's program 5 for `main`, `red` and `green` (each with its JSON
and Markdown report), and an offline check of the same three builds on Antibody's recorded fixture, where bug
B's paid proof is public: it still breaks `fees_to_treasury` at `AcmeVault.sweepFees(address)` on `red`, and
breaks no rule on `main` and `green`.

## Publishing (waits for an OK)

Release `v0.5.0` of `Himess/antibody` must be published first: it carries the action
(`.github/actions/immune-memory`) and the `antibody` binaries the action checks and runs.

```
git init -b main && git add . && git commit -m "Acme Lend at its final fix, with immune memory on pull requests"
gh repo create Himess/antibody-immune-demo --public --source . --push
git checkout -b red/unguarded-fee-sweep main && git apply patches/red-unguarded-fee-sweep.patch
git commit -am "Let a keeper sweep the fees too" && git push -u origin HEAD && gh pr create --fill
git checkout -b green/pending-fees-view main && git apply patches/green-pending-fees-view.patch
git commit -am "Add pendingFees(), for dashboards" && git push -u origin HEAD && gh pr create --fill
```

To make commits 14 and 18 readable (and the red pull request red), disclose their tags (and commit 13's), from
an account with NEAR credit at near-antibody for the run, then relay the run's TDX quote (or let the T1
relayer do it):

```
antibody --account <you>.testnet admin call --contract bounty.antibody.testnet --method request_disclosure --gas-tgas 200 \
  --args '{"program_id":5,"tags":[{"round":0,"tag":"ae7e21c8af1b96023b13ae1daff4bcd1076b5b282e4f6ce6fb91e6fdae5f6de4"},{"round":0,"tag":"a74dddb61406f343a53af7298ef64be2f9334763dfcd23f17ca3b85547e53b8c"},{"round":0,"tag":"325703227266ea4d3828142c490819fbc957bef5065a5ce0daeb03ee75242199"}]}'
antibody --account <you>.testnet tee confirm --run <its run id> --tx <its transaction> --tx-signer <you>.testnet
```

Then re-run the checks on both pull requests.
