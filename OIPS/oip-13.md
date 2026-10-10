---
oip: 13
title: Public ledger and migration path
description: Defines the v1 transaction and block format, publication of the ledger on GitHub, replay, and conditions for moving on-chain.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: https://github.com/open-science-org/OIPs/pull/10
status: Draft
type: Standards Track
category: Core
created: 2026-10-02
requires: 10, 12, 14
---

## Abstract

In v1 the OSO ledger runs on one operator-run node. Every state change is a signed transaction; transactions are grouped into hash-chained blocks; and state is a deterministic function of the block log. The log is published to a public GitHub repository from the first block, with periodic state snapshots, so anyone can replay it and check every balance. This OIP defines the transaction envelope, block format, state hash, publication, replay rules, and the conditions and steps for moving the ledger to smart contracts later. It states the trust assumptions of v1 explicitly.

## Motivation

OSO wants openness from day one without the cost and complexity of a blockchain before real money is involved. A public, replayable log gives transparency now. Its limits must be stated plainly: the operator orders transactions and can delay or omit them, and GitHub history can be rewritten by administrators.

### Prior work

- [Technical design v0 (2018)](https://github.com/open-science-org/OSO/blob/898ee42ebeb9ea7248214fa7c508a318df144d5d/OSO_design_v0.pdf): options for tokens on Ethereum, a child chain or a native chain, and the Interplanetary Idea System's naming, storage and identity layers.
- [OSO: An Idea Platform v0.3 (2018), §6](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): smart contracts on Ethereum as a settlement layer, IPFS storage, and minimal use of the blockchain.
- [Proof of Idea v0.0 (2018), §4](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): the first implementation, two contracts on the Ethereum Ropsten testnet.
- [OIP-1: IPFS integration and platform UI](./oip-1.md): storing content off-chain and only its address on-chain.
- [idea-hub issues #24 and #26 (2020)](https://github.com/open-science-org/idea-hub/issues/24): the 2020 stack and its REST API ([#26](https://github.com/open-science-org/idea-hub/issues/26)).
- [idea-hub pull request #33 (2020)](https://github.com/open-science-org/idea-hub/pull/33): the unmerged Solidity contract.
- [admin issue #7 (2017)](https://github.com/open-science-org/admin/issues/7): a multisig wallet for OSO, and early caution about wallet contract bugs.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Transaction envelope

Every transaction MUST contain:

| Field | Meaning |
| --- | --- |
| `domain` | `{ protocol: "OSO", network: <network id>, version: <envelope version> }` |
| `type` | Transaction type, handled by a module (OIP-12) |
| `sender` | The signing identity's address |
| `nonce` | Per-sender counter, starting at 0, incremented by one per transaction |
| `delegation` | Optional reference to a delegation (OIP-10) |
| `custody` | `self` or `operator` (OIP-10) |
| `body` | Type-specific fields |
| `signature` | Signature over the typed-data hash of all fields above |

1. Signatures MUST use typed structured data hashing in the style of EIP-712, including `domain`.
2. A transaction MUST be rejected if its nonce is not exactly the sender's next nonce. This, with `domain`, prevents replay within and across networks.

### 2. Blocks

Each block MUST contain: block number, previous block hash, operator-recorded timestamp, a day label (`yyyy-mm-dd`, UTC), the ordered list of transactions, the Merkle root of the transactions, the resulting state hash, and the operator's signature.

1. The block hash MUST be the hash of all fields except the operator's signature.
2. The timestamp is the operator's claim and MUST NOT be used by state transitions. Time-based rules MUST use block numbers.
3. The day label is also an operator claim. Its only use in state transitions is as the origin date of versions registered in the block (OIP-14 section 3). Day labels MUST NOT decrease from one block to the next, and MUST be within one day of the timestamp.

### 3. State hash

The state hash MUST be a Merkle root over all state entries, sorted by key, using a canonical serialization defined in the reference implementation. Two implementations that agree on the log MUST produce the same state hash after every block.

### 4. Publication

1. Every block MUST be committed to a public git repository (GitHub in v1; the host is replaceable), one file per block, with a signed commit, within one round (OIP-14) of being produced.
2. A state snapshot MUST be committed at least once per round.
3. The latest block hash SHOULD be published at least once per round to independent places outside GitHub (for example, mirrors run by community members), so that a rewritten history can be detected.
4. The operator MUST NOT rewrite published history. A correction MUST be a new transaction.
5. The log is tamper-evident relative to published checkpoints and independent mirrors. It MUST NOT be described as immutable.

### 5. Replay

1. A replay tool MUST rebuild state from the genesis block using only the published log, module code identified by code hash (OIP-12), and recorded inputs. It MUST make no network calls.
2. Replaying the full log MUST reproduce every block's recorded state hash.

### 6. Recorded inputs

1. AI outputs (pre-screen reports, suggested weights, domain similarities) and human decisions on them MUST enter the ledger as transactions, together with the model identifier, prompt or configuration identifier, and source references used.
2. Replay MUST read these records and MUST NOT re-run models or fetch live metadata. Re-running AI is an audit, separate from replay.
3. Imported metadata MUST be recorded as it was at import time.

### 7. Trust assumptions in v1

Implementations and interfaces MUST disclose that in v1:

- the operator orders transactions and can delay or omit them;
- the operator's choice of transactions influences block hashes used as selection seeds (OIP-11);
- the operator sets block day labels, which fix the origin dates of new submissions (OIP-14);
- the operator holds keys for users in operator custody (OIP-10);
- the operator runs the central content store, so it can read submissions before they are admitted and could remove stored files; removals are recorded on the ledger (OIP-16 section 5a);
- GitHub administrators can rewrite repository history; detection depends on mirrors and checkpoints.

### 8. Migration to smart contracts

1. Migration MUST NOT happen before milestone M3: a legal entity exists, outside funding is involved, and users need self-custody.
2. Before migration, settlement load (OIP-14 section 9) MUST be measured and the contract implementation MUST settle in bounded steps.
3. Migration steps:
   1. Publish the target contracts and an equivalence test suite that replays the v1 log against them.
   2. Choose a cutover block and announce it at least one round in advance.
   3. Freeze the v1 ledger at the cutover block and publish the final state hash.
   4. Load a snapshot of identities, ideas, balances, escrow, pending liabilities and claim slots into the contracts.
   5. Verify that the loaded state matches the final state hash, and publish the verification.
   6. Switch writes to the chain.
4. A later Core OIP MUST specify the target chain, contract authorization, key custody after migration, randomness, and upgrade governance.

## Rationale

- **GitHub first.** Transparency and replay are possible now at almost no cost; a chain adds trustlessness, which matters once real money and self-custody are involved.
- **Nonces and domains.** Typed signing alone does not stop a signed transaction from being replayed; nonces and a network domain do.
- **No clock in state transitions.** Timestamps are operator claims; block numbers are part of the verified log.
- **Recorded AI inputs.** Model outputs are not reproducible over time; recording them makes replay exact.
- **Explicit trust assumptions.** Following the 2026 technical review, v1 claims only what it provides.

### Open questions

1. Canonical serialization format for transactions and state.
2. Block interval and round length.
3. Who runs independent mirrors in v1?

## Backwards Compatibility

New. Replaces the storage and blockchain sections of the OSO Idea Platform whitepaper v0.3 (§6) for v1.

## Test Cases

To be added. Required cases: a transaction with a reused or skipped nonce is rejected; a transaction signed for another network is rejected; deleting the state database and replaying the published log reproduces the latest state hash; replay succeeds with networking disabled.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Operator censorship or reordering | Disclosed trust assumption; public log; migration path at M3 |
| History rewrite on GitHub | Signed commits, independent mirrors and published checkpoints |
| Replay of signed transactions | Per-sender nonces and network domain |
| Non-reproducible AI outputs | Recorded as inputs; replay never re-runs models |
| Failed or partial migration | Equivalence tests, announced cutover, frozen final state hash, published verification |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
