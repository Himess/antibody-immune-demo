# Local results (2026-11-07)

`antibody immune` (built from the Antibody repository's `phase-d-immune` branch) run locally against each
build, with view calls on NEAR testnet and local verifier runs only: no transaction was sent.

## Against program 5 on testnet

| Build | Exit | Commit 13 (A) | Commits 14 (B), 18 (F), 19 (G) | `AcmeVaultImpl` code under test | Reports |
|---|---|---|---|---|---|
| `main` (the final fix) | 0 | fixed; the TEE's receipt reproduced | not re-run: sealed | `0x8281ee18…`, the code round 2's release permits installed | `main.md`, `main.json` |
| `red` (bug B back) | 0 | fixed; the TEE's receipt reproduced | not re-run: sealed | `0x6d31f9a8…` | `red.md`, `red.json` |
| `red`, `--strict` | 3 | fixed | sealed, counted as inconclusive | `0x6d31f9a8…` | `red-strict.md`, `red-strict.json` |
| `green` (a new view) | 0 | fixed; the TEE's receipt reproduced | not re-run: sealed | `0xed2f383c…` | `green.md`, `green.json` |

- The build is the code that shipped: from this repository's artifacts and the mapping's constructor
  arguments, `AcmeLendFinal` and `AcmeVaultFixed` hash to `0x7deb43a5…` and `0x8281ee18…`, the code hashes
  the program's round-2 release permits installed on Sepolia and Base Sepolia.
- `red` cannot go red here yet: the proof of bug B (commit 14) is sealed until it is disclosed (README) or
  read with the project key, and bug #1's proof (commit 13), the one readable today, is still fixed on `red`.

## Offline, on the recorded fixture

Antibody's recorded fixture (`fixtures/acme-local`: the same contracts on a local chain, every proof public)
holds bug B's paid proof. The same three builds, from this repository's artifacts with the fixture
deployment's constructor arguments, were re-run against every recorded proof with the functions `antibody
immune` uses (`build`, the code under test, `rerun`), in a one-off local test:

| Build | `AcmeVaultImpl` code | B (bug #2, paid) | A, D (bug #1) | F (bug #3) | E, KNOWN (#4, the declared known issue) |
|---|---|---|---|---|---|
| `main` | `0x389bcf63…`, the fixture's round-2 code | fixed | fixed | fixed | still breaks `share_price` |
| `red` | `0xb7cc1385…` | **still breaks `fees_to_treasury` at `AcmeVault.sweepFees(address)`** | fixed | fixed | still breaks `share_price` |
| `green` | `0x3b6ab180…` | fixed | fixed | fixed | still breaks `share_price` |

The known issue is not fixed by design and is not a paid proof: the check on program 5 does not count it.
The raw output is `fixture-check.txt`.
