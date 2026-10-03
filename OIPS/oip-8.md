---
oip: 8
title: Token structure
description: Defines OSO supply and minting, the mint split and escrow, stakes, and per-idea IDEA tokens.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Core
created: 2026-10-02
---

## Abstract

OSO uses three separate instruments. **OSO** is the single platform currency: it is minted when an idea passes validation and pays for stakes, bounties, validator work and funding. **IDEA tokens** are issued per idea and are the ownership shares of that idea's retained income. **Reputation** is non-transferable and sets voting weight; its computation is specified in [OIP-9](./oip-9.md).

This OIP specifies OSO units, supply and minting, the split and escrow of each mint, submission stakes, the bootstrap allocation, and the structure of IDEA tokens. How value moves between ideas after it is minted or paid in (the payout waterfall) is specified in [OIP-14](./oip-14.md). In v1 all amounts are non-redeemable simulated units on the local ledger; IDEA transfers are disabled.

## Motivation

The earlier OSO papers describe tokens in several places, with gaps between them:

- Proof of Idea v0.0 mints OSO per validated idea and splits it 50/30/15/5, but does not specify escrow, eligibility, or what cited publications receive.
- The Idea Platform whitepaper v0.3 introduces IDEA tokens as "divisible ownership in the promise of something from an idea", but does not say how they relate to the owners' shares.
- The OSO v1 design doc gave retained value to owners and also gave IDEA holders a claim on incoming value. A technical review (2 Oct 2026) pointed out that these two rights compete for the same pool.

Implementations of the v1 ledger need one exact rule set: every payment MUST produce the same balances on every implementation, and every unit MUST be accounted for.

This OIP does not cover reputation ([OIP-9](./oip-9.md)), identity and key custody ([OIP-10](./oip-10.md)), validation and challenges ([OIP-11](./oip-11.md)), community parameters ([OIP-12](./oip-12.md)), idea attribution and value flow ([OIP-14](./oip-14.md)), or the ledger format and on-chain migration ([OIP-13](./oip-13.md)).

### Prior work

- [Proof of Idea v0.0 (2018), §1, §2.4 and §2.5](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): no pre-mined assets; minting on validation with a supply cap of 10^12 OSO and reward rate 10^-10; the 50/30/15/5 reward split.
- [OSO white paper (2017), §3.3 and §3.6](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_white_paper.pdf): what voting and tokens mean, and the costs of publication (review, idea, storage).
- [OSO: An Idea Platform v0.3 (2018), §3.3 and §4.3.3](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): the OSO utility token and per-idea IDEA tokens for "divisible ownership in the promise of something from an idea".
- [Technical design v0 (2018)](https://github.com/open-science-org/OSO/blob/898ee42ebeb9ea7248214fa7c508a318df144d5d/OSO_design_v0.pdf): three options for where tokens live (Ethereum, a child chain, a native chain) and idea wallets owned by researchers.
- [GIP attack-vector questions](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/AttackVectorQuestions.md): how new researchers acquire OSO, and whether a company could buy influence with tokens.
- [OIP-3: Funding OSO](./oip-3.md): the risk of token concentration in a few funders.
- [idea-hub pull request #33 (2020)](https://github.com/open-science-org/idea-hub/pull/33): an unmerged Solidity contract with an ERC-20 token, idea registration, validation and publication.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### Terminology

| Term | Meaning |
| --- | --- |
| Base unit | The smallest indivisible amount of OSO. 1 OSO = 10^18 base units. |
| bps | Basis points; 10,000 bps = 100%. |
| Idea | A registered work with a stable work ID and immutable version IDs ([OIP-16](./oip-16.md)). |
| Idea account | The ledger account that receives value for an idea before distribution. |
| Payout graph | Approved parent links that may move value ([OIP-14](./oip-14.md)). |
| α (alpha) | The retention rate of an idea, in bps. |
| Incoming value | OSO credited to an idea: a mint's cited share, funding, gifts, or value passed up from a child. |
| Pending liability | An amount owed but not yet settled, recorded on the ledger. |
| Round | A fixed number of blocks after which all pending liabilities settle ([OIP-14](./oip-14.md)). |

### 1. Arithmetic

1. All amounts MUST be integers in base units. Shares, weights and α MUST be integers in bps.
2. Every division MUST round down (floor). The destination of every rounding residue is specified in the section where the division occurs.
3. Every operation MUST conserve value: debits equal credits plus escrow plus pending liabilities.

### 2. Accounts

The ledger MUST keep integer balances for these accounts:

- **User accounts**, one per identity.
- **Idea accounts**, one per work ID.
- **Claim-slot accounts**, one per imported author slot, holding value for authors who have not yet claimed.
- **Unresolved-claims accounts**, one per imported idea whose author split is not yet adjudicated.
- **The OSO fund**, the platform treasury.
- **Escrow accounts**, one per pending mint.
- **The bootstrap pool**, created at genesis.

### 3. OSO supply and minting

1. At most T = 10^12 OSO MUST ever be minted.
2. When the n-th eligible idea passes validation, the ledger MUST mint `M_n = floor( (T − S_n) / R_inv )` base units, where T and S_n are in base units, S_n is the total OSO minted before it (including the bootstrap pool), and R_inv = 10^10 (a mint rate R = 10^-10, written as an integer divisor so that no fractional arithmetic is needed).
3. Burned OSO MUST NOT return to mintable supply.
4. Only newly submitted original work is eligible, and only once, when its first version passes validation. The ledger MUST NOT mint for imported works, metadata edits, ownership changes, status changes, or re-validation of a work that has already minted.
5. Revisions, reviews and replications MUST NOT mint until a later OIP specifies otherwise.

### 4. Mint split and escrow

1. Each mint M MUST be split as follows, each share computed as `floor(M × bps / 10,000)`:

   | Share | bps | Recipient |
   | --- | --- | --- |
   | Authors | 5,000 | The new idea's IDEA holders at the block of minting |
   | OSO fund | 3,000 | The OSO fund |
   | Cited ideas | 1,500 | The new idea's approved parents, as incoming value under [OIP-14](./oip-14.md), by approved weight |
   | Validators | 500 | Equally to every validator who voted before the deadline, whatever their vote |

2. Within a share, amounts MUST be computed as follows, each with floor rounding:
   - Authors: to each IDEA holder `floor(share × units / 1,000,000)`.
   - Cited ideas: to each approved parent j `floor(share × w_j / 10,000)`, as incoming value at j.
   - Validators: `floor(share / number of on-time validators)` each.
3. All rounding residue, from the split and from every division within a share, MUST go to the OSO fund.
4. If the new idea has no approved parents, the cited share MUST go to the OSO fund.
5. The whole mint MUST be held in escrow until the idea is Published under [OIP-11](./oip-11.md): its challenge window of W blocks has closed with no challenge or appeal pending. No part of it MAY be spent or moved before then.
6. When the idea is Published, every share MUST be released in the same block.
7. If the idea is Removed under OIP-11 (a challenge upheld and any appeal decided), the whole escrowed mint MUST be burned and the submission stake slashed (section 8).

### 4a. Validator fees

1. Each validator who votes before the deadline on an admission, challenge or appeal decision under OIP-11 MUST receive a fee of f_val OSO from the OSO fund, whatever their vote and whatever the outcome.
2. If the OSO fund cannot pay the fee, the decision MUST NOT start until it can; the idea waits (OIP-11 Waiting state).
3. The validators' mint share (section 4) is paid in addition to the fee.

### 5–7. Value flow

The payout waterfall, payout-graph rules and dust rules of earlier drafts are now specified in [OIP-14](./oip-14.md). Section numbers 8 to 12 are kept so that references from other OIPs stay valid.

### 8. Submission stakes

1. Each submission MUST lock a stake of s_sub OSO from the submitter.
2. The stake MUST stay locked until the routing decision is final under OIP-11. It MUST be returned when the idea is Published or Withdrawn, or when it is Returned and the appeal window has closed or the appeal has been decided against admission.
3. The stake MUST be slashed when the idea is Removed, or when it is RejectedSpam and the appeal window has closed or the appeal has been decided against admission. A share r_ch of a stake slashed on a successful challenge goes to the challenger (OIP-11 section 5); the rest goes to the OSO fund.
4. Validators MUST NOT be required to stake in v1.

### 9. IDEA tokens

1. Every idea that can earn income MUST have exactly 1,000,000 IDEA units, created when the idea is registered. The supply is fixed; this OIP allows no further issuance.
2. For a new submission, units MUST be allocated to the owners in proportion to their signed shares (share in bps × 100 units).
3. For an imported idea, units MUST be held by its unresolved-claims account until adjudication allocates them to claim slots.
4. IDEA units are the only payout right on an idea's retained value and on the authors' share of its mint.
5. IDEA units MUST NOT carry attribution or intellectual-property rights. Those are recorded separately and do not move with the units.
6. Payments to IDEA holders MUST be in OSO.
7. In v1, IDEA units MUST NOT be transferable. Transfers are enabled only by a later OIP, after the acceptance checks have passed and legal review is complete.

### 10. Bootstrap allocation

1. The genesis block MUST create a bootstrap pool of B OSO and record its amount publicly. B counts toward T.
2. The pool MUST be used only for first participants' submission stakes, first validators' starting balances and an initial balance of the OSO fund for validator fees, through public, logged transfers. Starting expertise for the first validators (OIP-9 section 7) is recorded publicly in the genesis block alongside the pool.
3. There MUST be no other pre-mine.

### 11. Reputation

Reputation MUST NOT be transferable, delegable or purchasable, and MUST NOT be derived from token balances. It is the only input to voting weight. Its computation is specified in [OIP-9](./oip-9.md).

### 12. Parameters

| Parameter | Symbol | v1 value | Set by |
| --- | --- | --- | --- |
| Base units per OSO | — | 10^18 | Protocol |
| Supply cap | T | 10^12 OSO | Protocol |
| Mint rate | R = 1 / R_inv | 10^-10 (R_inv = 10^10) | Protocol |
| Mint split | — | 5,000 / 3,000 / 1,500 / 500 bps | Protocol |
| Challenge window | W | TBD blocks | Community |
| IDEA units per idea | — | 1,000,000 | Protocol |
| Submission stake | s_sub | TBD OSO | Community |
| Validator fee | f_val | TBD OSO per vote | Community |
| Bootstrap pool | B | TBD, disclosed at genesis | Founding team, once |

A parameter change MUST take effect from a stated block and MUST NOT apply retroactively.

## Rationale

### Separate OSO and IDEA tokens

OSO measures value across the whole network and is needed for stakes, bounties and funding. IDEA tokens let someone back one specific idea without holding a claim on everything else. Keeping them separate also confines the instrument most likely to be treated as a security to a feature that is disabled in v1.

### Minting on validation

Minting follows Proof of Idea: publishing earns tokens, reversing the pay-to-publish model. Because AI can generate plausible papers cheaply, protection sits at the validation gate: stakes, AI pre-screening, per-identity limits, and full escrow of the mint during the challenge window. Escrowing only the authors' share would let colluding submissions release the fund, parent and validator shares before a challenge succeeds.

The mint declines very slowly with R = 10^-10: about 99.99 OSO after one million ideas, about 90.5 OSO after one billion, and about 36.8 OSO after ten billion. The cap is a long-run ceiling rather than a near-term constraint.

### Validators paid regardless of vote

Paying validators by agreement with the majority rewards conformity. Validation is an admission check, so validators are paid for doing it on time; sanctions for admission decisions overturned on challenge belong to reputation.

### IDEA tokens as the ownership shares

Two designs were considered:

- **Option A (specified): IDEA units are the ownership shares.** A 60/40 co-authorship is 600,000 and 400,000 units. There is one payout right per idea, so owners and holders can never claim more than exists.
- **Option B: a separate IDEA pool.** A fixed fraction of retained value (for example 20%) goes to IDEA holders and owners split the rest. Owners keep a non-sellable stake, at the cost of two overlapping rights per idea and a pool fraction to govern.

Option A is specified because exact, verifiable accounting is the v1 goal.

### Open questions

1. Should R stay fixed, or change over time? Should the mint depend on anything besides remaining supply?
2. Should revisions, reviews and replications mint, and how much?
3. Values for W, s_sub, f_val and B.
4. Should the 50/30/15/5 split become a community parameter?
5. May owners ever issue more IDEA units, for example to raise funding?

## Backwards Compatibility

This OIP replaces the token generation and reward sharing sections of Proof of Idea v0.0 (§2.4–2.5) and the token section of the OSO Idea Platform whitepaper v0.3 (§3.3). Changes from those documents:

- The mint is held in escrow during a challenge window.
- The cited share enters parents as incoming value under OIP-14, so each parent's α applies.
- The payout waterfall, payout-graph rules and dust rules moved to OIP-14.
- IDEA tokens are defined as the ownership shares, with fixed supply.
- Imported works never mint.

No deployed system depends on the earlier rules.

## Test Cases

All amounts below are in whole OSO for readability; implementations use base units.

### Mint

The first validated idea mints 100 OSO (taking B = 0 for readability; with a bootstrap pool the first mint is slightly smaller), with 5 validators voting on time and parents P1 (6,000) and P2 (4,000). Each validator also receives the fee f_val from the OSO fund.

| Share | Amount |
| --- | --- |
| Authors | 50 |
| OSO fund | 30 |
| P1, incoming value | 9 |
| P2, incoming value | 6 |
| Each of 5 validators | 1 |
| **Total** | **100** |

All 100 OSO, and the submission stake, stay locked until the idea is Published.

### Conformance checks

An implementation conforms when:

- [ ] Every operation satisfies debits = credits + escrow + pending liabilities, in base units.
- [ ] Supply equals the bootstrap pool plus total minted minus total burned, and transfers conserve supply.
- [ ] For every idea, payouts to IDEA holders never exceed retained value plus the authors' mint share.
- [ ] Each work ID mints at most once; imports, edits and ownership changes never mint.
- [ ] No part of an escrowed mint moves before its window closes, and a successful challenge burns all of it.
- [ ] Replaying the same log gives identical balances and supply on independent implementations.

## Reference Implementation

None yet. The v1 ledger will provide one.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Spam submitted to farm mints | Submission stake, full escrow of the mint, burn on successful challenge; AI pre-screening and per-identity limits |
| Low-quality but admissible work submitted to farm mints | Not prevented by admission, which is not a quality check; per-identity limits cap it, and mints do not raise voting weight except through the cited share received by parents (OIP-9). MUST be re-examined before tokens carry outside value |
| Validators favouring admission to get paid | Per-vote fee paid whatever the outcome (section 4a) |
| Minting the same work twice | One mint per work ID, on first validation; revisions and imports do not mint |
| Routing the 15% cited share to one's own ideas | Parent links and weights require approval, and self-citations are flagged (OIP-14) |
| Owners and holders claiming more than exists | One payout right per idea; fixed IDEA supply |
| Rounding differences between implementations | Integer base units, floor rounding, residue to the OSO fund |
| Concentrated OSO or IDEA holdings | Holdings never give voting weight; no pre-mine beyond a disclosed bootstrap pool; IDEA transfers disabled in v1 |
| Wash trading or price manipulation of IDEA units | Not possible while transfers are disabled; MUST be addressed before they are enabled |
| Bribery or vote trading | Not prevented by token design; reputation-weighted voting is one defense |

**Legal considerations.** IDEA units that carry a share of an idea's income may be classified as securities, and OSO may be regulated depending on how it is issued and used. In v1 all units are non-redeemable simulated units, nothing is sold, and IDEA transfers are disabled. Before any real-money flow, a legal entity must exist and the token structure must receive legal review. This OIP makes no legal classification.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
