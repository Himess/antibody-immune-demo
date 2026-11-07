### Antibody immune memory: no paid proof re-run here breaks a rule (1 of 4 re-run, 3 sealed)

Program 5 on testnet (*Acme Lend v1 — pre-launch (testnet)*, `bounty.antibody.testnet/5`), anchored at chain 11155111 block 12063269. Each paid proof this run could read was re-run with the verifier, locally, at the program's anchor, against this build's code, under round 2's rules. Nothing was sent.

| Result | Commit | Paid for | Read through | Rule broken | Entry point |
|---|---|---|---|---|---|
| fixed | 13 by agent-a.antibody.testnet (round 0, Done:Paid) | solvency (critical), 0.1 wNEAR | published content key | - | - |
| not re-run (sealed) | 14 by agent-b.antibody.testnet (round 0, Done:Paid) | fees_to_treasury (high), 0.03 wNEAR | - | - | - |
| not re-run (sealed) | 18 by agent-f.antibody.testnet (round 0, Done:Paid) | solvency (critical), 0.1 wNEAR | - | - | - |
| not re-run (sealed) | 19 by agent-g.antibody.testnet (round 0, Done:Paid) | no key (gap claim), 0.01 wNEAR | - | - | - |

- Commit 14, not re-run (sealed): not disclosed, no published content key, and no project key here (a repository secret, never given to pull requests from forks)
- Commit 18, not re-run (sealed): not disclosed, no published content key, and no project key here (a repository secret, never given to pull requests from forks)
- Commit 19, not re-run (sealed): not disclosed, no published content key, and no project key here (a repository secret, never given to pull requests from forks)
- The TEE's receipt reproduced locally before the re-run (commit: match): 13 yes

<details><summary>The code under test: 9 targets, 2 from this build (as a round manifest, sha256 <code>58318e40cc0585b66d5b8c336cb3b42d5e8d9aaad03cba2c67592a63bbc859e9</code>)</summary>

| Target | Address | Code | From | Code hash |
|---|---|---|---|---|
| AcmeGuard | `0x0A1811b3a36287727abC7065AAcC8Fc4547b2f98` | anchor | - | `0xf45c264911a1eecaa48c7c7b191650c4b38d4cb1b04e3861b284d211c5686367` |
| acUSD | `0xF4E8D84A44d5d901B322D82E60b1a6cDc7620955` | anchor | - | `0x3afc4a0ac6cb2e7864dc0a31324dae2889b78d9f4bae5a8bddcd4158adf1c3ab` |
| acETH | `0x4EDD629802596011Ab8D76D996EF35b419961C76` | anchor | - | `0xbea2eab7997a15413a1e5f54c3c57369bf3e0aa935d477869ee18f38d8be57f0` |
| AcmePool | `0x0053feFbAab1D98b3cc6765f435Dbb6f8682f245` | anchor | - | `0xb1ecd0600348f3f1cf532e5ff1a4b1d976d0aa492e5a9d9ef1ea4882c17ae5c4` |
| AcmeLend | `0xCa1FAFbCFC14d665d76045A018B215987f39f8A7` | anchor | - | `0x2a09df94f5f2041dc6de562b1a2bb52a6429f165964cf0a369efb006513eee3b` |
| AcmeLendImpl | `0x4c348e8758DEEa78Df03D0C0b20c58D1148edc2c` | pull request | AcmeLendFinal (out/AcmeFinalFix.sol/AcmeLendFinal.json) | `0x7deb43a5fd8a6aac5aee95c158e0e552f6e866407b3f4c62442b39c6c6092adb` |
| AcmeVault | `0x0433CEB84dB807402DE6642929EbF1450381CC46` | anchor | - | `0x2a09df94f5f2041dc6de562b1a2bb52a6429f165964cf0a369efb006513eee3b` |
| AcmeVaultImpl | `0x0fD4fe9dA00079c3f3a94Be994482ab8eB8C44fa` | pull request | AcmeVaultFixed (out/AcmeFinalFix.sol/AcmeVaultFixed.json) | `0x6d31f9a83817f9ff2a902423ed7ea84399d2a79e2a36545f4af6dd4fa432855d` |
| AcmeTreasury | `0x25D16D1C0C5B86B88aDAE16e5A526A5526DC0C97` | anchor | - | `0xf9431cd77b16235470fe08f3ce041f6da901556130b38cc20ba616432b4e3ded` |

</details>

Exit code 0. 0: no re-run paid proof breaks a rule; 2: a paid proof still breaks one; 3: none breaks, but a re-run was inconclusive. The local verifier is antibody-verifier 0.1.0, the crate the TEE's pinned build (wasm sha256 ccc27f0ca60c…) is made from. Sealed proofs re-run inside the TEE are next, not built.
