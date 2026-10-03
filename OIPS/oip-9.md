---
oip: 9
title: Reputation and expertise
description: Defines per-domain expertise and integrity scores, and how they combine into voting weight.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Module
created: 2026-10-02
requires: 8, 11, 14
---

## Abstract

Voting weight in OSO comes from reputation, never from token balances. This OIP defines the reference Reputation module. A user's weight in domain D is their **expertise** in D, capped, multiplied by their **integrity**. Expertise measures contributions to D: ideas that others built on, reviews that held up, and endorsements weighted by the endorser's own expertise. Integrity measures fair conduct as a validator and reviewer, based only on decisions overturned on challenge and missed deadlines, never on scientific disagreement. All quantities are integers computed deterministically from the ledger, so anyone can recompute any score. Communities may tune the parameters or replace the module.

## Motivation

The 2018 Idea Platform whitepaper defined an expertise score e_D and a reputation score r_S but left their functions open. [OIP-8](./oip-8.md) requires that voting weight not derive from token holdings. Validation, review matching and governance all need a concrete, reproducible weight. Two failure modes shape the design: weight that can be bought, and weight that rewards agreeing with the majority.

### Prior work

- [OSO white paper (2017), §3.2 and §3.3](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_white_paper.pdf): expertise of an entity in a research domain (the e-value) and its role in voting.
- [OSO: An Idea Platform v0.3 (2018), §3.2](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): a user's identity, expertise score e_D and reputation score r_S, with their functions left open.
- [Technical design v0 (2018)](https://github.com/open-science-org/OSO/blob/898ee42ebeb9ea7248214fa7c508a318df144d5d/OSO_design_v0.pdf): researcher profiles with reputation and expertise scores, universal and per domain.
- [GIP attack-vector questions](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/AttackVectorQuestions.md): reputation as a vector of scores updated from a user's activity.
- [RR-index (2017)](https://github.com/open-science-org/RR-index/blob/a8fe298e76208bc411b28ffba5e014df55304004/README.md): a proposed domain-independent metric of research impact.
- [OIP-4: Validator merit](./oip-4.md): rewarding careful validators without rewarding conformity.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. General rules

1. Reputation MUST NOT be transferable, delegable or purchasable.
2. Every input MUST be read from the ledger. AI-derived inputs (such as domain similarity) MUST be the values recorded on the ledger, not recomputed.
3. All arithmetic MUST use integers. Fractions are in bps (10,000 = 1). `isqrt` is the integer square root (largest n with n² ≤ x).
4. Scores MUST be recomputed at the end of every round of K blocks (as defined in OIP-14) and MUST be constant within a round.

### 2. Inputs

| Input | Source |
| --- | --- |
| share(u, i) | u's owner share of idea i, in bps, from its signed version or adjudicated claim |
| sim(i, D) | Similarity of idea i to domain D, in bps; an AI suggestion recorded on the ledger and open to community correction |
| use(i) | Total OSO received by idea i **directly** as the cited share of its own children's mints (OIP-8 section 4), in whole OSO (base units divided by 10^18, rounded down). Value passed up the graph from further away, funding and gifts are all excluded. |
| age(x) | Blocks elapsed since event x |
| review rating | Ratings of a review by other users in the domain (OIP-17 section 10) |
| endorsement | A signed rating of user v by user u in domain D, from 0 to 10,000 bps |

Only direct cited shares of mints count, because value passed up the graph may have started as funding: counting it would let anyone raise a parent's owners' weight by funding one of its children. Mints themselves require an admitted child, so each unit of `use` stands for a validated piece of work that built on idea i.

### 3. Decay

`decay(x, h) = floor( 10,000 / 2^floor(age(x) / h) )`, where h is a half-life in blocks. The value halves once per full half-life.

### 4. Expertise

For user u and domain D:

```
E_ideas(u, D)   = Σ over ideas i owned by u:
                  share(u,i) × sim(i,D) × isqrt(use(i)) × decay(i, h_idea)    / 10^8
E_reviews(u, D) = Σ over reviews r written by u:
                  sim(r,D) × quality(r) × decay(r, h_review)                  / 10^8
E_endorse(u, D) = Σ over endorsements of u in D by v:
                  rating × min(E_prev(v, D), cap) × decay(endorsement, h_review) × k_endorse / 10^12
E(u, D)         = E_ideas + E_reviews + E_endorse
```

1. `quality(r)` is the mean of the review ratings recorded on review r ([OIP-17](./oip-17.md) section 10), which are whole numbers from 0 to 10, converted to bps as `floor( 1,000 × sum of ratings / number of ratings )`, excluding ratings by owners of the reviewed idea, so that critical reviews are not marked down by the people they criticize. A review with no counted ratings has quality 5,000.

   With these scales, one review of maximum quality and similarity is worth 10,000 points, and an idea fully owned and fully in D is worth 10,000 × isqrt(use) points: an idea whose children's mints have paid it 100 OSO counts like ten excellent reviews. An endorsement transfers at most k_endorse of the endorser's capped expertise.
2. `E_prev` is the expertise from the previous round. Using the previous round's value makes the recursion well defined and deterministic.
3. Imported ideas count toward E_ideas only after a claim is adjudicated, multiplied by the import discount `d_import` (bps).
4. A user's endorsements of themselves, and endorsements between co-owners of the same idea within the last h_review blocks, MUST NOT count.

### 5. Integrity

For user u:

```
I(u) = (upheld(u) + 1) × 10,000 / (decided(u) + 2)  −  penalty(u)      (floor at 0)
```

1. `decided(u)` is the number of u's votes (admission, challenge and appeal decisions under OIP-11) whose decision has become final: no challenge or appeal can still change it.
2. `upheld(u)` is the number of those votes that were not overturned. A vote is overturned only when the final decision went against it **because a later challenge or appeal reversed the decision it took part in** (OIP-11 section 5). Being in the minority of a decision that stood is never an overturn.
3. Disagreeing with other validators is never a violation.
4. `penalty(u)` is the sum of penalties for proven violations: p_deadline for each missed vote deadline, p_overturn for each overturned vote (in either direction: a wrongful admit or a wrongful not-admit), and p_misconduct for proven plagiarism or spam by u. Each penalty decays with h_idea.
5. A new user starts with I = 5,000.

### 6. Voting weight

```
weight(u, D) = min( E(u, D), cap ) × I(u) / 10,000
```

1. weight(u, D) MUST be the only reputation input in domain D. In validation it decides eligibility for the roster (OIP-11 section 3), and each drawn validator then has one vote. In review matching it ranks candidate reviewers. In governance votes it is the vote weight.
2. A user MAY have different weights in different domains.

### 7. Bootstrap

Before any user has earned expertise, the founding team MAY assign starting expertise to the first validators, recorded publicly in the genesis block alongside the bootstrap allocation (OIP-8 section 10). Bootstrap expertise MUST decay with h_review and MUST NOT exceed `cap / 2`.

### 8. Parameters

| Parameter | Meaning | v1 value |
| --- | --- | --- |
| h_idea | Half-life of idea credit and penalties | TBD blocks (target about 5 years) |
| h_review | Half-life of review and endorsement credit | TBD blocks (target about 2 years) |
| cap | Maximum expertise counted toward weight | TBD |
| d_import | Discount on claimed imported ideas | 5,000 bps |
| k_endorse | Share of an endorser's capped expertise one endorsement can transfer | 1,000 bps |
| p_deadline, p_overturn, p_misconduct | Penalties | TBD |

All parameters are community parameters (OIP-12).

## Rationale

- **Expertise × integrity.** The 2018 whitepaper separated expertise (what you know) from reputation (how you behave). Multiplying them means neither a deep record nor clean behaviour alone gives large weight.
- **Only direct mint shares count as impact.** Counting funding, or value passed up from funded descendants, would let money buy voting weight indirectly.
- **Square root.** It limits the dominance of a single highly used idea and is exactly computable in integers, unlike a logarithm.
- **Previous-round endorsements.** A fixed-point (PageRank-style) iteration would also work, but depends on iteration count and rounding. Using last round's values gives one exact answer per round.
- **Integrity without conformity.** Following the 2026 technical review, integrity is based on decisions overturned on challenge and on deadlines, never on agreeing with the majority.
- **Step decay.** It is exact in integers and easy to verify; smoother decay can be specified later.

### Open questions

1. Values for the half-lives, cap and penalties.
2. Should the cap be global or per domain?
3. How are domains defined and split as the graph grows?
4. Should replications count as a separate expertise input?

## Backwards Compatibility

Replaces the informal expertise and reputation formulas in the OSO Idea Platform whitepaper v0.3 (§3.2). The whitepaper's vote value, which combined staked tokens with expertise, is dropped: tokens no longer add voting weight.

## Test Cases

To be added with the reference implementation. Required cases: a user with no record has weight 0; funding an idea does not change E for its owners or for the owners of any ancestor; an overturned vote lowers I in either direction; ratings by the reviewed idea's owners do not change a review's quality; disagreeing with a majority decision that is upheld does not lower I; two independent computations from the same log give identical weights for every user and domain.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Buying weight through funding | Only direct cited shares of children's mints count toward use(i); funding, gifts and value passed up the graph do not |
| Farming mints to raise a parent's use(i) | Each mint needs an admitted child; per-identity submission limits; square root; self-citations flagged (OIP-14). Not fully prevented: low-quality but admissible children still count |
| Citation rings inflating use(i) | Parent links and weights require approval; self-citations flagged for approvers (OIP-14) |
| Endorsement rings | Self and recent co-owner endorsements excluded; endorsements weighted by endorser expertise and scaled by k_endorse; cap |
| Authors punishing critical reviewers | Ratings by the reviewed idea's owners excluded from quality(r) |
| Sybil accounts | New accounts have no expertise and neutral integrity; identity rules in OIP-10 |
| Entrenchment of early users | Decay; cap; bootstrap expertise capped and decaying |
| Bribery or vote trading | Not prevented by scoring; public votes make patterns detectable |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
