---
oip: 17
title: Peer review
description: Defines one simple, open peer-review process for v1, and the principles any later community-specific process must keep.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Module
created: 2026-10-03
requires: 9, 11, 16
---

## Abstract

Validation ([OIP-11](./oip-11.md)) only admits an idea to the graph; peer review judges its quality. The OSO Idea Platform whitepaper lets each sub-network and channel run its own review process, provided every process is open, selects reviewers fairly and transparently, keeps reviewing after publication, and never hides rejected work. This OIP records those four principles as rules for any review process, and then specifies one simple process that every community uses in v1: three reviewers invited for each admitted idea and paid from the OSO fund, open reviews from anyone qualified at any time, one score and a recommendation per review, and a rating that keeps updating. Reviews are themselves ideas ([OIP-16](./oip-16.md)). Community-specific review processes come later.

## Motivation

Open, perpetual review is OSO's second pillar, but the v1 drafts covered it in a few lines of OIP-11. The v1 reviewer queue needs exact rules, and OIP-9 needs a defined source for the review ratings it uses in expertise scores. Starting with one simple process lets the pilot test review before communities customize it.

### Prior work

- [OSO white paper (2017), §3.5 and §3.6.1](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_white_paper.pdf): an open review process in two stages, initial review and perpetual review; paid reviewers; quality measured on several dimensions (reproducibility, readability, citations, importance); rejected work published with its reasons rather than hidden; ratings that change as work is reproduced or fails to be.
- [Proof of Idea v0.0 (2018), §1 and Figure 2](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): publishing as layers of curation: layer 1 spam filter (now OIP-11), layer 2 peer review, layer 3 public (perpetual) review, layer 4 sub-networks with their own review rules.
- [OSO: An Idea Platform v0.3 (2018), §1, §4.3.2 and §5](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): decentralized peer review in which a review is itself an idea; sub-networks and channels with their own review policies; and four characteristics every such process must keep: open by default, fair and transparent reviewer selection, perpetual review, and no hard reject.
- [OIP-6: IdeaBoard](./oip-6.md): a discussion board as a first version of public review.
- [OIP-7: Publishing as a cascade of TCRs](./oip-7.md): each review layer as a curated list with its own incentives.
- [GIP attack-vector questions](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/AttackVectorQuestions.md): reviewing as a way for new researchers to earn OSO.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Terminology

| Term | Meaning |
| --- | --- |
| Review | An idea of type `review` targeting exactly one version of another idea (OIP-16 section 2) |
| Invited review | A review written by an invited reviewer, and paid |
| Open review | A review written by anyone eligible without an invitation; unpaid in v1 |
| Rating | The current combined score of a version |
| Review rating | A score given to a review itself, used for the reviewer's expertise (OIP-9) |

### 2. Principles for every review process

Any review process in OSO, including later community-specific ones, MUST:

1. **Be open.** Reviews, scores and review ratings are signed and public.
2. **Select reviewers fairly and transparently.** Selection follows published, reproducible rules with conflict exclusions; authors do not choose their reviewers.
3. **Be perpetual.** Reviews can be added at any time after publication, and ratings keep changing as they arrive.
4. **Hide nothing.** There is no hard reject: negative reviews, low ratings and their history stay public alongside the idea.

### 3. Scope in v1

1. In v1, every community MUST use the process in sections 4 to 10. Communities MAY set its parameters (section 11) but MUST NOT replace it.
2. A later OIP will let sub-networks and channels define their own review processes as replaceable modules ([OIP-12](./oip-12.md)), for example invited panels, more reviewers or extra scoring dimensions. Any such process MUST keep the principles in section 2.

### 4. Eligibility and conflicts

1. A user MAY review in domain D only if their weight in D (OIP-9 section 6) is at least the community's minimum reviewer weight.
2. The conflict rules of OIP-11 section 3.2 apply. A user MUST NOT review an idea they own, or rate a review of an idea they own.

### 5. Reviewer selection

The default selection method is a **weighted random draw with a load cap**. The alternatives considered, with their pros and cons, are in the Rationale.

1. **Candidates.** When an idea becomes Admitted (OIP-11), the candidates are the users who meet section 4 in the idea's domain and have fewer than `max_open` accepted reviews still in progress.
2. **Selection weight.** Each candidate's selection weight is their weight in the domain (OIP-9 section 6). OIP-9 already caps expertise, so no single expert dominates.
3. **Draw.** Sort the candidates by address. For draw j = 1, 2, …, compute `r_j = hash(seed ‖ version ID ‖ j) mod W`, where seed is the hash of the block in which the idea was admitted and W is the total selection weight of the candidates still in the list. Walk the sorted list adding selection weights; the first candidate at which the running total exceeds `r_j` is drawn and removed from the list. Repeat until k candidates are drawn or the list is empty. A candidate's chance of being drawn is proportional to their weight, and anyone can recompute the result.
4. **Replacement.** An invitation not accepted within the acceptance window, or accepted but not delivered by the review deadline, MUST be replaced by continuing the draw (the next j) over the remaining candidates. A reviewer who accepts and misses the deadline receives the deadline penalty in OIP-9.
5. **Too few candidates.** If fewer than k candidates exist, the invitations already drawn proceed, and the remaining invitations MUST wait and be drawn as candidates become available. Eligibility and conflict rules MUST NOT be relaxed.
6. **Disclosure.** The candidate list, weights and seed MUST be recorded with each draw. In v1 the operator can influence the seed (OIP-11 section 3.4); this MUST be disclosed and the draw MUST NOT be described as manipulation-resistant.

### 6. Invited reviews

1. When an idea becomes Admitted, the core MUST invite k reviewers for its current version, selected under section 5.
2. Each invited review delivered on time MUST be paid f_rev OSO from the OSO fund. The payment MUST NOT depend on the score, the recommendation or how the review is rated. If the fund cannot pay, invitations wait until it can.
3. Authors never pay for review.
4. Review payments are payments to people for tasks; they do not enter the payout waterfall (OIP-14 section 8).

### 7. Open reviews

Any eligible user MAY review any Admitted or Published version at any time, subject to section 4. Open reviews are unpaid in v1 and count toward the rating like invited reviews.

### 8. Review content

1. A review MUST be registered as an OIP-16 idea of type `review` whose `targets` names the reviewed version. Its content MUST cover: what the work claims, whether the evidence supports the claims, whether the results could be reproduced from the methods, code and data provided, and what should change.
2. Each review MUST carry, in the same transaction:
   - a **score**, a whole number from 0 to 10, for the overall quality of the work, using these anchors:

     | Score | Meaning |
     | --- | --- |
     | 0 | Fundamentally flawed: the claims are not supported at all |
     | 3 | Major problems: key claims are weakly supported or cannot be checked |
     | 5 | Sound, with significant limitations |
     | 7 | Solid: claims supported, minor issues |
     | 10 | Excellent: claims well supported, reproducible, and a significant contribution |

     Whole numbers between the anchors are allowed.
   - a **recommendation**: `endorse`, `revise` or `concerns`.
3. AI assistance MUST be declared in the review's `ai_disclosure` (OIP-16 section 4). The signing reviewer is responsible for the whole review.
4. A question that does not apply to an idea (for example reproducibility for a `hypothesis`) MAY be answered "not applicable" with a reason. A community MAY add review questions in its setup; it MUST NOT remove the required ones in v1.
5. The owners MAY reply by registering a review of the review.
6. A `revise` recommendation MAY lead the owners to submit a new version, under OIP-11 section 6. A new version starts with no reviews of its own; earlier reviews stay visible in its history.

### 9. Ratings

1. A version's rating MUST be the weighted median of its review scores, each weighted by the reviewer's weight in the domain (OIP-9 section 6) at the round the review was recorded. Sort scores ascending (ties by reviewer address) and take the first score at which the running total of weights reaches at least half of the total weight.
2. The rating MUST be recomputed whenever a review is added, and its history MUST be kept.
3. Replications of the version (OIP-16 type `replication`) MUST be shown next to the rating as counts of each outcome.
4. In v1, ratings MUST NOT change any balance, mint, weight set or α.

### 10. Review ratings

1. Any user with weight greater than zero in the domain MAY rate a review once, with a whole number from 0 (useless) to 10 (careful and very useful), subject to section 4.2. The rating judges the review's care and usefulness, not agreement with it.
2. OIP-9's `quality(r)` is the mean of these ratings, excluding ratings by owners of the reviewed idea (OIP-9 section 4).

### 11. Parameters

| Parameter | Meaning | v1 value | Set by |
| --- | --- | --- | --- |
| k | Invited reviewers per admitted idea | 3 (a community MAY lower it to 2 if its reviewer pool is too small; the setting is public) | Community |
| f_rev | Payment per invited review, from the OSO fund | TBD OSO | Community |
| Minimum reviewer weight | Weight needed to review in a domain | TBD | Community |
| Acceptance window | Blocks to accept an invitation | TBD | Community |
| Review deadline | Blocks to deliver after accepting | TBD | Community |
| max_open | Accepted reviews a reviewer may have in progress before being skipped | 3 | Community |
| Selection method | How invited reviewers are drawn | Weighted random draw (section 5) | Protocol in v1 |

## Rationale

- **Principles now, customization later.** The Idea Platform whitepaper lets each sub-network and channel run its own review, within four shared characteristics. Writing those characteristics down now keeps later community processes compatible, while one process for v1 keeps the first build and the pilot simple.
- **Review after admission.** Proof of Idea separated a cheap spam filter (layer 1) from expert review (layer 2) and perpetual public review (layer 3). Ideas enter the graph quickly; review continues.
- **Paid reviews from the fund.** The 2017 white paper's "paid and quality peer review" should not depend on authors paying, which would bring back pay-to-publish.
- **Three reviewers.** With two, the median is just one of the two scores, so a single harsh or generous reviewer decides the rating. With three, one outlier cannot. More cost more and strain small reviewer pools; open reviews can add further opinions at no cost to the fund.
- **Pay that ignores the verdict.** Otherwise honest criticism is punished.
- **A 0 to 10 scale.** Reviewers cannot meaningfully use a finer scale, and whole numbers keep the median exact. Anchors at 0, 3, 5, 7 and 10 make scores comparable across reviewers. The rating is shown on the same 0 to 10 scale.
- **One score.** The 2017 white paper measured quality on several dimensions (reproducibility, readability, importance). One score plus written comments is enough to start; dimensions can be added once the pilot shows which ones reviewers use.
- **Weighted median.** It resists a few extreme or coordinated scores, and the weights come from reputation, which cannot be bought.
- **No value from ratings in v1.** Tying money to an untested rating system would invite manipulation.

### Reviewer selection: options considered

| Option | How it works | Pros | Cons |
| --- | --- | --- | --- |
| **Weighted random draw (default)** | Random among eligible reviewers, with probability proportional to reputation weight; load cap | Favors expertise without always picking the same people; reproducible; authors cannot choose; hard to predict who will review, so harder to bribe in advance | Experienced reviewers still get more requests (eased by the load cap); a weak reviewer can still be drawn; depends on reputation being accurate early on |
| Uniform random draw | Random among everyone above the minimum weight | Simplest; spreads work evenly; no reliance on reputation above the threshold | Ignores expertise above the minimum, so a barely qualified reviewer is as likely as a leading expert |
| Top-k by weight | The k highest-weight eligible reviewers | Most expert reviewers every time | The same few people review everything; predictable, so easy to target; overloads them; entrenches incumbents |
| Topic matching by similarity | AI compares the submission with each reviewer's own ideas and invites the closest matches, as conference systems such as the Toronto Paper Matching System and OpenReview's affinity scores do | Best topical fit, especially in broad domains | Depends on an AI model (needs recorded inputs and two models under OIP-15); authors can tune wording to steer matches; tends to pick close collaborators and competitors |
| Bidding | Reviewers choose which ideas to review, as in many computer-science conferences | Motivated reviewers with real interest | Friends can bid on each other's work; collusion rings that coordinated bids have been reported at major conferences (Littman, Communications of the ACM, 2021) |
| Editor or channel chair assigns | A named person picks reviewers, as in journals | Human judgment of fit and balance | Centralized, opaque and open to bias, the problems OSO set out to fix; a possible option for channels later, with assignments public |
| Author-suggested reviewers | Authors name reviewers | Authors know who understands the work | Strong conflicts of interest; publishers have retracted papers after author-suggested reviewer contacts turned out to be fake. Not allowed |

The weighted draw is the default because it keeps the two properties the whitepaper requires, fair and transparent selection, while still sending ideas to people with relevant expertise. Two refinements are likely later: combining it with topic matching (for example, weighting by expertise × similarity once similarity is a recorded, two-model AI input), and letting channels use an editor-assigned panel with public assignments. Both would come through the later OIP on community review processes (section 3.2).

### Open questions

1. Should ratings ever affect value, for example a mint bonus for reproduced work?
2. Which scoring dimensions should be added first?
3. Should review bounties from authors, channels or funders be allowed in v1, beyond the k fund-paid reviews?
4. When should communities be able to replace the review process, and through which OIP?

## Backwards Compatibility

Takes over peer review from OIP-11, which keeps only the rules for new versions. Implements the review layers of Proof of Idea (layers 2 and 3), §3.5 of the 2017 white paper and §5 of the Idea Platform whitepaper, with one network-wide process in place of per-community processes for now. Unlike the 2017 white paper, acceptance is not decided by a majority of reviewers.

## Test Cases

### Weighted median

Scores and weights on one version: A 8 (weight 3), B 2 (weight 1), C 6 (weight 2). Sorted: B, C, A. Total weight 6, half is 3; running totals 1, 3. The rating is 6.

### Required cases

- [ ] An admitted idea gets k invitations drawn by section 5, excluding conflicts and reviewers at max_open.
- [ ] The same seed, candidates and weights always give the same draw.
- [ ] Over many draws, a candidate with twice the weight is drawn about twice as often.
- [ ] An invited review delivered on time is paid f_rev whatever its score.
- [ ] An open review counts toward the rating and is not paid.
- [ ] An owner cannot review their own idea or rate reviews of it.
- [ ] Adding a review changes the rating, and the old rating stays in its history.
- [ ] Ratings change no balance, weight set or α.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Low-effort reviews to collect payment | Required content; low review ratings lower the reviewer's expertise and eligibility (OIP-9) |
| Authors arranging friendly reviewers | Conflict rules; a random, reproducible draw that authors do not control; no bidding or author suggestions; all reviews public |
| Overloading the most expert reviewers | Load cap (max_open); capped expertise in OIP-9 |
| Retaliatory or biased reviews | Signed, public reviews; owners may reply; weighted median limits one reviewer's effect |
| Coordinated scores | Weighted median; weights from reputation that cannot be bought; minimum reviewer weight |
| Authors punishing critical reviewers | Pay independent of the verdict; owners' ratings of reviews excluded from quality(r) |
| Draining the OSO fund | k paid reviews per admitted idea only; admission requires a stake and validation |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
