---
oip: 10
title: Ownership and identity
description: Defines identities, key custody, ownership of ideas, imported-author claims and AI-produced work.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Core
created: 2026-10-02
requires: 8, 11
---

## Abstract

This OIP defines who can act on the OSO ledger and who owns what. An identity is a key pair plus attestations, such as an ORCID sign-in, an institutional email, or peer vouches. Ideas record three separate things: attribution (who is credited), payout rights (IDEA units, [OIP-8](./oip-8.md)), and license (intellectual-property terms). New ideas need every owner's signature. Imported papers are registered without owners and give each listed author a claim slot that is settled through an adjudication process. AI agents can sign actions on behalf of an identity but cannot own ideas.

## Motivation

The 2017 URI proposal and the 2018 design assumed that a key could be tied to a real researcher. ORCID only proves control of an ORCID account, OpenAlex author matches can be wrong, and imported papers have authors who never agreed to OSO's terms. The ledger needs exact rules so that credit and money go to the right people and can be corrected.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Identities

1. An identity MUST be a secp256k1 key pair. Its identifier is the Ethereum-style address of the public key.
2. An identity MAY be a person or an organization. An AI agent MUST NOT be an identity.
3. Identities carry **attestations**, each recorded as a ledger transaction:

   | Attestation | Issued by | Proves |
   | --- | --- | --- |
   | `orcid` | The operator, after ORCID sign-in | Control of an ORCID account |
   | `email-domain` | The operator, after email verification | Control of an address at an institution's domain. Only domains on the community's published list of institutional domains qualify; addresses at general mail providers do not produce this attestation |
   | `vouch` | Another identity | That identity's statement that the key belongs to the named person |

4. Attestations MUST NOT be described as proof of real-world identity. Interfaces SHOULD show which attestations an identity has.
5. The operator MUST rate-limit new identities. The limit is a community parameter.

### 2. Key custody in v1

1. After ORCID sign-in, the operator MUST create the user's key pair and, in v1, hold it on the user's behalf.
2. The operator MUST publish its custody policy before v1 is used by people outside the founding team. The policy MUST state who can access keys, how keys are recovered, and how a user can export their key and take custody.
3. Every action signed by the operator on a user's behalf MUST be recorded with a flag showing operator custody.

### 3. Delegated signing

1. An identity MAY delegate a limited signing permission to another key, for example an AI service. A delegation MUST name the delegate key, the allowed transaction types and an expiry block, and MUST be revocable.
2. Every transaction signed under a delegation MUST record the delegation it used.
3. AI services and ingestion pipelines MUST act this way: under a delegation from the operator's or another organization's identity, never as identities of their own (section 1.2). The delegating identity is accountable for what they submit.

### 4. Ownership of new ideas

1. An idea version MUST list its owners and their shares in bps, summing to 10,000.
2. A new submission MUST be signed by every listed owner. A submission missing a signature MUST NOT be accepted as Submitted (OIP-11) and so cannot enter validation.
3. Changing owners or shares MUST be done by a registry update signed by all current owners. The update MUST move the idea's IDEA units to match the new shares (share in bps × 100 units), so that owner shares and IDEA units never disagree while IDEA transfers are disabled. It MUST NOT create a new idea version and MUST NOT mint (OIP-8).
4. Each idea MUST record three separate fields:

   | Field | Meaning | Changes when |
   | --- | --- | --- |
   | Attribution | The people credited as authors | Only by a correction signed by all owners, or by adjudication |
   | Payout rights | IDEA units (OIP-8) | By allocation, by an ownership change under item 3, and, once enabled, by transfer |
   | License | Terms for reuse, for example CC-BY-4.0 | Only by all owners, and never retroactively |

   A change to one field MUST NOT change the others. Owner shares and payout rights are the same field in v1: owner shares are recorded as IDEA units.

### 5. Registration time

The block containing a version's registration gives an operator-recorded registration time. Interfaces MUST NOT present it as independent proof of priority.

### 6. Imported works

1. A work imported from an outside source (OpenAlex, arXiv or similar) MUST be registered with its source identifiers and author list as attribution only. It MUST carry no owners, no signatures and no financial terms.
2. Each listed author MUST get a separate **claim slot**, recording the source's author identifier (ORCID, OpenAlex ID) where one exists.
3. The idea's IDEA units MUST be held by its unresolved-claims account (OIP-8 section 9) until adjudication.
4. A **claim** is a transaction by an identity asserting that it is the author of a slot. A claim whose ORCID attestation matches the slot's ORCID MAY be approved automatically after a public notice period. All other claims, and any disputed claim, MUST go to adjudication.
5. **Adjudication** decides who holds each slot and how the idea's IDEA units are split among the slots. In v1, adjudication is done by a disclosed panel named by the founding team. Its decisions MUST be recorded with reasons and MUST be open to appeal.
6. Shares MUST NOT be assumed equal unless the adjudication records equal shares as its decision.
7. Value accrued to an unresolved idea or slot MUST remain an identifiable liability on the ledger.

### 7. AI-produced work

1. Every idea MUST record an `ai_disclosure` field: none, assisted, or generated, with a free-text description.
2. Ownership of AI-produced work belongs to the identity that submitted it. Communities MAY apply different α or reward rules to work marked generated (OIP-12).

## Rationale

- **Attestations, not proof.** Following the 2026 technical review, ORCID and email checks prove control of accounts, not identity. Making each check a named attestation lets communities decide which they require.
- **Operator custody in v1.** It lets researchers sign in with ORCID and never manage keys, which was a major barrier in 2018. Disclosure and an export path keep it honest until self-custody (M3).
- **Three fields.** Selling IDEA units must not make the buyer an author, and a license change must not move money.
- **Claim slots.** Bibliographic data is often wrong; per-author slots and adjudication avoid paying the wrong person and keep unclaimed value visible.

### Open questions

1. Who should sit on the v1 adjudication panel, and how is it replaced later?
2. Length of the notice period for automatic ORCID claims.
3. Should organizations need a different attestation set?

## Backwards Compatibility

Replaces the Unique Researcher Identity proof of concept (2017) and the identity section of the OSO Idea Platform whitepaper v0.3 (§3.2 and §6.4).

## Test Cases

To be added. Required cases: a submission missing one owner's signature never enters validation; an ownership change moves IDEA units to match the new shares, and does not mint or create a version; a contested ORCID match goes to adjudication; value to an unresolved slot remains a liability until adjudication.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Impersonating an author of imported work | Notice period, adjudication for any mismatch or dispute, appeal |
| Sybil identities | Rate limits; voting weight from reputation (OIP-9), which new identities lack |
| Operator misuse of custodied keys | Published custody policy, operator-custody flag on every signed action, key export |
| Compromised AI-service key | Scoped, expiring, revocable delegations |
| Attribution changed to redirect money | Attribution and payout rights are separate fields |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
