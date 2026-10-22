# Genesis Block Puzzle (P341) — Full Solution

A complete, independently reconstructed solution to the 2026 Bitcoin genesis-block
multi-signature puzzle. The puzzle's prize was **already swept** on 2026-10-17 by a
third-party buyer (`bc1qdu8m…qzad`, block 971803); this repository reconstructs and
publishes the key material for **verification and educational purposes only**.

> ⚠️ **Security note.** The private keys below correspond to an address whose funds
> were already claimed on-chain. They control **no live funds**. Do not send anything
> to that address, and do not reuse these keys anywhere. They are published solely so
> that anyone can re-derive and verify the on-chain witness signatures.

## Address & structure

- **Puzzle address:** `bc1qfkhx02v89u2qyyyljeczw6hu9sr437y44t7ae5yf09thrdukfqesnjg2wj`
- **Script:** `OP_2 <PUB1> <PUB2> OP_2 OP_CHECKMULTISIG` (2-of-2 P2WSH)
- **Witness program (sha256 of script):** `4dae67a9872f1402109f9670276afc2c0758f895aafddcd089795771b7964833`

## The recipe (both co-creators)

| Field | PUB1 | PUB2 |
|---|---|---|
| cut window | `GB[262:400]` (The Times headline hex) | same |
| entropy | `SHA256(cut)[:16]` | same |
| mnemonic | `scan amateur fit scrap light over fire flip cushion reject hurt ugly` | same |
| passphrase | `Satoshi Nakamoto` | `Satoshi Nakamoto` |
| **PBKDF2-HMAC-SHA512 rounds** | **181505** (genesis time 18:15:05 UTC) | **200913** |
| account | `1231006505` (genesis block `time`) | `1231006505` |
| path | `m/48'/0'/1231006505'/2'/0/0` | `m/48'/0'/1231006505'/2'/0/0` |
| pubkey | `027b497b1d82986ebe11bca71108ca43446c00edb069cc839638a0b76529130906` | `033614b02e8e3aeeed9e14ac67839f85d54b694eafd91ca07f1afeb06510832ec1` |
| **privkey** | `60d09998a128dcd2642cc7f16db24f1525cf18564e976a3e87fa5e48e24874e9` | `4b18979d14cb062679c4b16a9b04bd163138720aa4ffc4c6245e9cb8203b93c6` |
| master fingerprint | `0852014c` | `040e2791` |

## Verification

Six independent checks confirm the reconstruction (see `PUB2_SOLUTION.md` §"六重独立证据"):
master-fingerprint reproduction, control-group for PUB1, PUB2 pubkey hit, end-to-end
address derivation, on-chain witness script byte-match, and on-chain BIP143 signature
validation against both pubkeys.

## Why this matters / how it was missed

The puzzle's real entropy + passphrase were never published; the author sold them
encrypted (BIE1) to the winning bidder. The non-standard PBKDF2 round counts
(181505 / 200913) are not 2048, which is why standard-tool brute force failed. The
breakthrough was an exhaustive C scan over 433 genesis-block windows × round counts
1–2,100,000, hitting 200913 at window 262.

## Files

- `PUB2_SOLUTION.md` — full write-up, evidence, and derivation details (the canonical result file).
- `LICENSE` — MIT.

This repository contains no executable solver, no private infrastructure, and no
third-party secrets — only the publicly reconstructable solution and its on-chain proof.
