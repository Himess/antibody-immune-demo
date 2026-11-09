# Antibody immune memory: a demo

Acme Lend is a fictional lending protocol with planted bugs, a **deliberately vulnerable test fixture** that
demonstrates [Antibody](https://github.com/Himess/antibody): never deploy it to mainnet. Its program on
near-antibody (NEAR testnet, program 5, anchored at Sepolia block 12063269) paid for four proofs; two rounds of
fixes followed, and the final fix shipped through the release gate.

This repository is Acme Lend at that final fix, as it shipped on testnet: `AcmeLendFinal` and
`AcmeVaultFixed` (`src/acme-fixes`) over the contracts they fix (`src/acme`), with the compiler settings it was
deployed with. Its build reproduces the code the release permits installed, byte for byte.

**Read the check named "enclave: every paid proof (the verdict)".** It re-runs every paid proof, sealed ones
included, inside the enclave, and takes about 11 minutes, one pull request at a time. The check named "runner
(advisory)" is green whenever no proof it can read breaks, and sealed proofs are not readable on a runner, so
on the red pull request it stays green while the verdict fails. The latest runs (v0.6.1, 2026-11-09):
[PR #2, red](https://github.com/Himess/antibody-immune-demo/actions/runs/42538787235) and
[PR #1, green](https://github.com/Himess/antibody-immune-demo/actions/runs/42538785818).

Every pull request gets two checks (Antibody's
[docs/immune.md](https://github.com/Himess/antibody/blob/main/docs/immune.md#re-runs-in-the-enclave)):

- **`enclave` ("enclave: every paid proof (the verdict)"), in the enclave:** the pull request's build is proposed as the next round of **program 12**, a
  program kept for these checks (Acme Lend at its own addresses, with paid proofs of its own). near-antibody
  re-runs every paid proof, sealed or not, inside the enclave, each re-run attested on NEAR, and the check reads
  which keys still break: red if one does. The job holds two scoped keys and runs none of the pull request's
  code: the project's (`propose_round` and the manifest's upload) and a bot's (`open_round`, so the next pull
  request finds the slot free).
- **`immune` ("runner (advisory)"), on the runner:** each paid proof of **program 5** (the recorded run's program) that the check can
  read is re-run by the local verifier against the pull request's build. It is the only check a pull request
  from a fork gets, since a fork gets no secret; its `enclave` check then fails with "not checked in the
  enclave".

## What is here

| Path | What |
|---|---|
| `.antibody/immune.toml` | The mapping: program 5, and the program target each contract builds (`AcmeLendFinal` replaces the `AcmeLendImpl` implementation, `AcmeVaultFixed` the `AcmeVaultImpl` one), with their constructor arguments as deployed |
| `.antibody/proofs/` | The sealed blobs of the paid commits 13, 14, 18 and 19, public on chain since their reveal (read with read-only RPC calls) |
| `.antibody/keys/13.hex` | Commit 13's content key, already published in the Antibody repository (`fixtures/zk/a`) |
| `.github/workflows/immune.yml` | The two checks, on `pull_request` (never `pull_request_target`): `immune`, `round` (builds the pull request's round manifest, with no secret) and `enclave` |
| `patches/` | Two pull requests, prepared as patches |
| `results/` | The check run locally against each build (view calls and local verifier runs only) |

## The paid proofs of program 5 (the check on the runner)

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

| Branch | Change | `enclave` (program 12) | `immune` (program 5, on the runner) |
|---|---|---|---|
| `red/unguarded-fee-sweep` | Drops the admin check from `AcmeVaultFixed.sweepFees`: bug B comes back | **red**: the paid proof of bug B (commit 56) breaks `fees_to_treasury` (high) again | green until commit 14's proof is readable (disclosure); then red, naming the entry point |
| `green/pending-fees-view` | Adds a view, `AcmeVaultFixed.pendingFees()`: new runtime code, same behaviour | **green**: every paid key fixed | green |

Measured on testnet (2026-11-08), both pull requests checked at once: PR #2 took the slot (round 6, re-run
133 breaks slot 1 again), PR #1 queued and its manifest was renumbered from round 6 to 7 (every re-run
fixed). Every run and transaction:
[docs/results/immune-demo-2026-11-08.json](https://github.com/Himess/antibody/blob/main/docs/results/immune-demo-2026-11-08.json).

## Local results

`results/README.md` has the runs against testnet's program 5 for `main`, `red` and `green` (each with its JSON
and Markdown report), and an offline check of the same three builds on Antibody's recorded fixture, where bug
B's paid proof is public: it still breaks `fees_to_treasury` at `AcmeVault.sweepFees(address)` on `red`, and
breaks no rule on `main` and `green`.
