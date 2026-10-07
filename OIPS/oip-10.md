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

This OIP defines who can act on the OSO ledger and who owns what. An identity is a key pair plus attestations, such as a linked ORCID account, an institutional email, or peer vouches. Users sign in with common accounts (Google, email and others) and add attestations later; what an identity may do depends on its attestations and earned reputation, not on how it signed in. Ideas record three separate things: attribution (who is credited), payout rights (IDEA units, [OIP-8](./oip-8.md)), and license (intellectual-property terms). New ideas need every owner's signature. Imported papers are registered without owners and give each listed author a claim slot that is settled through an adjudication process. AI agents can sign actions on behalf of an identity but cannot own ideas.

## Motivation

The 2017 URI proposal and the 2018 design assumed that a key could be tied to a real researcher. ORCID only proves control of an ORCID account, OpenAlex author matches can be wrong, and imported papers have authors who never agreed to OSO's terms. The ledger needs exact rules so that credit and money go to the right people and can be corrected.

### Prior work

- [Unique Researcher Identity (URI, 2017)](https://github.com/open-science-org/URI/blob/f4b7526c754eda8413ece85a71f63c0bac9e4adf/README.md): a blockchain-based researcher identity, possibly using third-party identity services and oracles to pull ORCID data.
- [Technical design v0 (2018)](https://github.com/open-science-org/OSO/blob/898ee42ebeb9ea7248214fa7c508a318df144d5d/OSO_design_v0.pdf): mapping real-world researchers to OSO identities, and ideas jointly owned with percentage shares.
- [OSO: An Idea Platform v0.3 (2018), §3.2 and §6.4](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): identity as a key pair, a username and a real-world identity, and the acknowledged difficulty of linking them.
- [Proof of Idea v0.0 (2018), §2.1 and §5](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): a list of verified users, genesis users, and ways to register new ones.
- [OIP-2: Funding application](./oip-2.md): using grant agencies' verified applicants as an early identity source.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Identities

1. An identity MUST be a secp256k1 key pair. Its identifier is the Ethereum-style address of the public key.
2. An identity MAY be a person or an organization. An AI agent MUST NOT be an identity.
3. Identities carry **attestations**, each recorded as a ledger transaction:

   | Attestation | Issued by | Proves |
   | --- | --- | --- |
   | `orcid` | The operator, after the user links an ORCID account | Control of an ORCID account |
   | `email-domain` | The operator, after email verification | Control of an address at an institution's domain. Only domains on the community's published list of institutional domains qualify; addresses at general mail providers do not produce this attestation |
   | `vouch` | Another identity | That identity's statement that the key belongs to the named person |

4. Attestations MUST NOT be described as proof of real-world identity. Interfaces SHOULD show which attestations an identity has.
5. The operator MUST rate-limit new identities. The limit is a community parameter.
6. **Sign-in methods.** A user signs in with any method the operator supports, for example a Google account, a one-time link sent to an email address, a GitHub account or an ORCID account. Other providers, such as an Apple account, MAY be added later. Signing in proves control of that external account and lets the user act as their identity; on its own it is not an attestation and grants no trust.
7. A user MAY link several sign-in methods to one identity. Linking a new method MUST require being signed in with a method already linked, so that a second account does not create a second identity by accident. Unlinking MUST leave at least one method.
8. The ledger MUST NOT record a user's email address or sign-in account identifiers in plain form. Interfaces MUST NOT show them publicly unless the user chooses to.

### 1a. What an identity may do

| Action | Requires |
| --- | --- |
| Browse, search, chat within the free allowance | Nothing; no sign-in needed to browse |
| Submit ideas, write open reviews, rate reviews | Any sign-in, within stricter rate limits for identities with no attestation |
| Claim an imported work | An `orcid` attestation matching the slot for automatic approval; otherwise adjudication with evidence (section 6) |
| Be drawn as a validator or invited reviewer | Earned weight in the domain (OIP-9), and any attestations the community's setup requires (for example `orcid` or `email-domain`) |
| Receive real-money payouts (after M3) | Attestations and checks set by the later legal and custody rules |

Communities MAY require more for an action, but MUST NOT require less than this table.

### 2. Key custody in v1

1. On a user's first sign-in, with any supported method, the operator MUST create the user's key pair and, in v1, hold it on the user's behalf.
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
- **Easy sign-in, earned trust.** Requiring ORCID to sign in would exclude engineers, designers and funders, and adds friction for researchers. Any common account can sign in; trust comes from attestations and reputation, so creating many easy accounts gains little: they cannot validate, review by invitation or claim works.
- **Operator custody in v1.** It lets people sign in with an account they already have and never manage keys, which was a major barrier in 2018. Disclosure and an export path keep it honest until self-custody (M3).
- **Three fields.** Selling IDEA units must not make the buyer an author, and a license change must not move money.
- **Claim slots.** Bibliographic data is often wrong; per-author slots and adjudication avoid paying the wrong person and keep unclaimed value visible.

### Open questions

1. Who should sit on the v1 adjudication panel, and how is it replaced later?
2. Length of the notice period for automatic ORCID claims.
3. Should organizations need a different attestation set?
4. Which sign-in methods to support at launch, and whether to use a hosted sign-in service or run our own.

## Backwards Compatibility

Replaces the Unique Researcher Identity proof of concept (2017) and the identity section of the OSO Idea Platform whitepaper v0.3 (§3.2 and §6.4).

## Test Cases

To be added. Required cases: a submission missing one owner's signature never enters validation; an ownership change moves IDEA units to match the new shares, and does not mint or create a version; a contested ORCID match goes to adjudication; a second sign-in method links to the same identity only when added while signed in; an identity with no attestation cannot be drawn as a validator; value to an unresolved slot remains a liability until adjudication.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Impersonating an author of imported work | Notice period, adjudication for any mismatch or dispute, appeal |
| Sybil identities from easy sign-in accounts | Rate limits, stricter for identities with no attestation; submission stakes; trusted roles need attestations and earned reputation (section 1a), which new identities lack |
| Takeover of a linked sign-in account | Linking requires an existing method; users see and can remove linked methods; key export and custody policy (section 2) |
| Operator misuse of custodied keys | Published custody policy, operator-custody flag on every signed action, key export |
| Compromised AI-service key | Scoped, expiring, revocable delegations |
| Attribution changed to redirect money | Attribution and payout rights are separate fields |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
