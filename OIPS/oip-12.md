---
oip: 12
title: Module interface and community setups
description: Defines how replaceable modules plug into the OSO core and how communities choose modules and parameters.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Core
created: 2026-10-02
requires: 8, 10, 13, 14
---

## Abstract

OSO is a small fixed core with replaceable modules. This OIP defines the core's boundary, the manifest every module must declare, how modules interact only through core events, and the deterministic dispatch order. It also defines **community setups**: public, versioned documents in which a sub-network or channel chooses module implementations and parameter values. Setups activate at a stated block, never apply retroactively, and can be forked.

## Motivation

Between 2018 and 2020, OSO's storage layer changed three times (IPFS, OrbitDB, libtorrent) and each change meant starting over. Different research communities also need different rules: validator counts, review policies, retention rates. A fixed module interface lets parts be replaced and customized without breaking the protocol or other communities.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. The core

The core consists of:

1. The idea format: stable work ID, immutable version IDs, the registry fields of OIP-10.
2. The transaction envelope and block format (OIP-13).
3. Accounts, OSO supply and the mint (OIP-8), and the payout graph and waterfall (OIP-14).
4. The module interface and dispatcher defined here.

Changes to the core require a Core OIP. Everything else is a module.

### 2. Module manifest

Every module MUST publish a manifest on the ledger before it can be used:

| Field | Meaning |
| --- | --- |
| `id` | Unique name, for example `oso.validation.random-n` |
| `version` | Semantic version |
| `kind` | The slot it fills: identity, edge-weights, pre-screen, validation, review, value-flow, minting, reputation, governance, storage, ai-service, or other |
| `code_hash` | Hash of the exact implementation |
| `handles` | Transaction types the module processes |
| `state` | The state namespace it owns |
| `emits` | Event types it emits |
| `consumes` | Event types it reacts to |
| `parameters` | Each parameter's name, type, bounds and default |
| `spec` | The OIP that specifies it |

### 3. Module rules

1. A module MUST read and write only its own state namespace and core state exposed to it through the interface.
2. A module MUST NOT read another module's state. Modules interact only by emitting and consuming core events.
3. A module MUST be deterministic: same state and inputs give the same outputs. It MUST NOT use clocks, randomness other than the block seed, network calls or floating-point arithmetic.
4. A module that moves OSO MUST do so through the core's accounting functions, which enforce conservation.
5. A module MUST NOT mint or burn except through the core's mint and burn functions, under the rules of OIP-8.

### 4. Dispatch order

Within a block, the core MUST process transactions in block order. For each transaction:

1. The core verifies the envelope (signature, nonce, delegation).
2. The handling module, chosen by the transaction type and the sender's community setup, processes it.
3. Events emitted are delivered to consuming modules in ascending module `id` order, then by emission order. Events emitted while handling events are queued and delivered the same way, up to a depth limit. Exceeding the depth limit is a failure.
4. If any step fails, the whole transaction MUST be reverted: no state change, no events, and the nonce still consumed. A failed transaction stays in the block, marked failed.

### 4a. Block-end processing

Some rules fire when a number of blocks has passed, with no transaction to trigger them. After the last transaction of every block, the core MUST run these steps in this order, each over its items in ascending idea work ID (then ascending validator address where relevant):

1. Deadline expiries: validators whose vote deadline (OIP-11) ended at this block are replaced.
2. Decision closings: votes, challenges and appeals whose collection is complete or whose deadline ended are decided (OIP-11).
3. Window closings: ideas whose challenge window or appeal window ended at this block move on (OIP-11), and escrow and stakes are released, burned or slashed (OIP-8).
4. Settlement: queued value-flow transfers are processed, and at a round's last block all owed amounts are queued first (OIP-14 sections 9 and 10).
5. Score recomputation: at a round's last block, reputation scores are recomputed (OIP-9).

Each step runs as the core, consumes no nonce, emits events delivered as in section 4, and is recorded in the block so that replay reproduces it.

### 5. Community setups

1. A sub-network or channel MUST have a setup document recorded on the ledger, listing for each slot a module `id`, `version` and parameter values within the manifest's bounds.
2. A setup MUST have a version number and an activation block. A new version MUST NOT take effect before its activation block and MUST NOT change outcomes of earlier blocks.
3. Who may change a setup is defined by the community's governance module.
4. Anyone MAY **fork** a setup: create a new community whose setup starts as a copy. Ideas are global; a forked community can list existing ideas, but payouts already made are unchanged, and existing ideas stay governed by the setup they were submitted to (item 5). A fork governs only new submissions to it. Moving an existing idea to another community requires a later OIP.
5. An idea is governed by the setup of the community it was submitted to, at the version active in each block.

### 6. Reference modules for v1

| Slot | Reference module | Specified in |
| --- | --- | --- |
| identity | ORCID and attestations | OIP-10 |
| validation, review, pre-screen | Random-N admission and expertise-matched review | OIP-11 |
| reputation | Expertise × integrity | OIP-9 |
| value-flow, minting | Fixed core behaviour in v1 | OIP-14, OIP-8 |
| storage | Hashes and links only | — |
| governance | Founding team, with advisory votes from M3 (see OIP-0, Adoption) | — |

In v1, value flow and minting are core behaviour with community-set parameters (α, challenge window, stakes), not replaceable modules.

## Rationale

- **Events instead of shared state.** If modules read each other's state, replacing one breaks the others.
- **Block-end processing.** Escrow release, deadlines and settlement depend on elapsed blocks, not on any one sender, so the core runs them at a fixed point in each block. This keeps replay deterministic without trigger transactions that someone would have to remember to send.
- **Ascending-id event order.** A fixed order is required for identical replays; any fixed order works, and this one needs no extra configuration.
- **Value flow in the core for v1.** Exact accounting across communities is easier to guarantee with one implementation; making it replaceable can come later.
- **Forking.** Following OIP-5, the remedy for a captured community is a fork that lists the same ideas and governs new work under its own rules. It cannot change the rules for ideas already submitted elsewhere; that limit is deliberate, so that no one can escape flow-back obligations by forking.

### Open questions

1. Depth limit for nested events.
2. Should communities be able to replace the minting split, and within what bounds?
3. How are new module versions reviewed before communities can adopt them?
4. Should owners be able to move an idea to a forked community, and on what terms?

## Backwards Compatibility

New. Generalizes the sub-network and channel ideas in the OSO Idea Platform whitepaper v0.3 (§1) and the custom validation layers of OIP-5.

## Test Cases

To be added. Required cases: a module reading another module's state is rejected; a failing transaction reverts all effects but consumes the nonce; block-end steps run in the specified order and replay identically; a setup change does not alter earlier results; two implementations dispatching the same block give the same state hash.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| A malicious module draining funds | All OSO movement through core accounting functions; mint and burn only through core |
| Non-deterministic modules breaking replay | Determinism rules; code hash in the manifest |
| Capture of a community's setup | Public versioned setups, activation delay, forking |
| Event loops | Depth limit |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
