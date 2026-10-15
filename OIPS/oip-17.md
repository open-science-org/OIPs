---
oip: 17
title: Peer review
description: Defines one simple, open peer-review process for v1, and the principles any later community-specific process must keep.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: https://github.com/open-science-org/OIPs/pull/10
status: Draft
type: Standards Track
category: Module
created: 2026-10-03
requires: 9, 11, 16
---

## Abstract

Review in OSO is a **pipeline of layers that each community configures**, because review serves different purposes and fields need different mixes. Each layer states who takes part, how they are selected, what a review contains, what it pays, and which rating it feeds. The reference module offers layer types: a spam filter (validation, [OIP-11](./oip-11.md)), assigned expert review (drawn and paid), expert reader review (any expert, any time), community review (any interested member), AI agent review (registered AI agent identities, [OIP-10](./oip-10.md) section 1b) and invited panels. The v1 default pipeline is the first four, with an expert rating and a community rating shown separately; AI agent review can be turned on by a community and feeds its own AI rating. Every configuration must keep the principles of the Idea Platform whitepaper: open, fair and transparent selection, perpetual, and nothing hidden. Reviews are themselves ideas ([OIP-16](./oip-16.md)).

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
| Assigned review | A review by a reviewer drawn and invited for the idea (layer 2); paid |
| Expert reader review | A review by a qualified expert who chose to review the idea (layer 3); unpaid in v1 |
| Community review | A review by any signed-in OSO member interested in the idea (layer 4); unpaid |
| Layer | One step of a community's review pipeline (section 2a) |
| Expert rating | The current combined score of a version from assigned and expert reader reviews |
| Community rating | The current combined score of a version from community reviews, shown separately |
| AI rating | The combined score from AI agent reviews, where that layer is enabled, shown separately |
| Review rating | A score given to a review itself, used for the reviewer's expertise (OIP-9) |

### 2. Principles for every review process

Any review process in OSO, including later community-specific ones, MUST:

1. **Be open.** Reviews, scores and review ratings are signed and public.
2. **Select reviewers fairly and transparently.** Selection follows published, reproducible rules with conflict exclusions; authors do not choose their reviewers.
3. **Be perpetual.** Reviews can be added at any time after publication, and ratings keep changing as they arrive.
4. **Hide nothing.** There is no hard reject: negative reviews, low ratings and their history stay public alongside the idea.

### 2a. Review pipelines

Review serves different purposes: keeping spam out, checking quality, keeping judgement current as work is used, and hearing from everyone who cares about an idea. No single process serves them all, and different fields need different mixes. So review is a **pipeline of layers** that each community configures.

1. A community's review module (OIP-12) MUST be configured as an ordered list of **layers** in its setup. Each layer states:
   - **who** takes part (a role or eligibility rule, for example drawn experts, any expert, any member, registered AI agents, an invited panel);
   - **how** participants are selected (drawn, self-selected, invited by a named editor, or all eligible);
   - **when** it runs (at admission, at a fixed time after, continuously) and how many reviews it seeks;
   - **what** a review must contain (required questions, score scale, recommendation);
   - **pay**, if any, and from which account;
   - **which rating** its reviews feed, and how scores within it are combined.
2. Every layer MUST keep the principles in section 2. Layers that use drawing or invitation MUST record who was selected and why.
3. Each rating MUST be shown separately and labelled by its layer or group of layers. A community MAY also show a headline rating computed from the others, but only with published weights, and the separate ratings MUST remain visible.
4. In v1, ratings MUST NOT change any balance, mint, weight set or α, whatever the configuration.

**Layer types in the reference module**

| Layer type | Who | Purpose | Default pay | Feeds |
| --- | --- | --- | --- | --- |
| Spam filter | Drawn validators (OIP-11) | Admission: not spam, meets checkable criteria | Per-vote fee (OIP-8) | Admission only |
| Assigned expert review | k experts drawn per admitted idea (section 5) | A careful, timely check of every admitted idea | Yes (section 6) | Expert rating; expertise (OIP-9) |
| Expert reader review | Any expert in the domain, any time (section 7) | Ongoing expert judgement as work is used and reproduced | No | Expert rating; expertise |
| Community review | Any signed-in member (section 7) | Feedback and interest from the wider community | No | Community rating |
| AI agent review | Registered AI agent identities (OIP-10 section 1b), section 7a | Fast, broad checks: statistics, citations, code that runs, consistency of claims and evidence | None, or to the agent's operator if the community chooses | AI rating; agent reputation (OIP-9 section 6a) |
| Invited panel | Reviewers chosen by a named editor, with choices and reasons public | Journal-style review for a channel | Community choice | A channel rating |
| Automated verification | Pinned, deterministic tools run by the operator: proof checkers (for example Lean), code re-execution, data checksums (section 7b) | Machine-checkable evidence that a proof checks or results reproduce | None | Verification results shown with the ratings; not a rating |

New layer types are added as module versions (OIP-12).

**v1 default pipeline:** spam filter, assigned expert review (k = 3), expert reader review and community review, with an expert rating and a community rating. AI agent review is available but off by default; a community turns it on in its setup.

### 3. Scope in v1

1. In v1, communities configure their pipeline from the layer types above and the parameters in section 11. The rules in sections 4 to 10 apply to the layer types they describe.
2. A later OIP may add layer types or let communities replace the review module entirely (OIP-12). Any configuration MUST keep the principles in section 2.

### 4. Eligibility and conflicts

1. A user MAY write an assigned or expert reader review in domain D only if their weight in D (OIP-9 section 6) is at least the community's minimum reviewer weight.
2. Any signed-in user MAY write a community review (OIP-10 section 1a), within a per-identity limit per round.
3. The conflict rules of OIP-11 section 3.2 apply to assigned and expert reader reviews. No user MAY review an idea they own, in any layer, or rate a review of an idea they own.

### 5. Reviewer selection

The default selection method is a **weighted random draw with a load cap**. The alternatives considered, with their pros and cons, are in the Rationale.

1. **Candidates.** When an idea becomes Admitted (OIP-11), the candidates are the users who meet section 4.1 and 4.3 in the idea's domain and have fewer than `max_open` accepted reviews still in progress.
2. **Selection weight.** Each candidate's selection weight is their weight in the domain (OIP-9 section 6). OIP-9 already caps expertise, so no single expert dominates.
3. **Draw.** Sort the candidates by address. For draw j = 1, 2, …, compute `r_j = hash(seed ‖ version ID ‖ j) mod W`, where seed is the hash of the block in which the idea was admitted and W is the total selection weight of the candidates still in the list. Walk the sorted list adding selection weights; the first candidate at which the running total exceeds `r_j` is drawn and removed from the list. Repeat until k candidates are drawn or the list is empty. A candidate's chance of being drawn is proportional to their weight, and anyone can recompute the result.
4. **Replacement.** An invitation not accepted within the acceptance window, or accepted but not delivered by the review deadline, MUST be replaced by continuing the draw (the next j) over the remaining candidates. A reviewer who accepts and misses the deadline receives the deadline penalty in OIP-9.
5. **Too few candidates.** If fewer than k candidates exist, the invitations already drawn proceed, and the remaining invitations MUST wait and be drawn as candidates become available. Eligibility and conflict rules MUST NOT be relaxed.
6. **Disclosure.** The candidate list, weights and seed MUST be recorded with each draw. In v1 the operator can influence the seed (OIP-11 section 3.4); this MUST be disclosed and the draw MUST NOT be described as manipulation-resistant.

### 6. Assigned expert reviews

1. When an idea becomes Admitted, the core MUST invite k reviewers for its current version, selected under section 5. These are the layer-2 assigned reviews.
2. Each assigned review delivered on time MUST be paid f_rev OSO from the OSO fund. The payment MUST NOT depend on the score, the recommendation or how the review is rated. If the fund cannot pay, invitations wait until it can.
3. Authors never pay for review.
4. Review payments are payments to people for tasks; they do not enter the payout waterfall (OIP-14 section 8).

### 7. Expert reader and community reviews

1. **Expert reader reviews.** Any user meeting section 4.1 MAY review any Admitted or Published version at any time. Expert reader reviews are unpaid in v1 and count toward the expert rating and the reviewer's expertise exactly like assigned reviews.
2. **Community reviews.** Any signed-in user MAY write a community review of any Admitted or Published version, within the per-identity limit. Community reviews count toward the community rating only. They do not count toward the expert rating or toward anyone's expertise (OIP-9).
3. **Routing.** Reviews of every layer are registered without validation (OIP-11) and without a submission stake. Like any idea, a review MAY be challenged for spam or plagiarism under OIP-11 section 5; a review found to be spam or plagiarism is marked as such and stops counting toward any rating.
4. Every review records its layer; interfaces MUST show the layer next to each review.

### 7a. AI agent reviews

1. Where a community enables the AI agent layer, any registered AI agent identity (OIP-10 section 1b) that meets the layer's rules MAY review Admitted or Published versions, or agents MAY be selected by the layer's selection rule (for example a draw weighted by agent reputation).
2. An agent review MUST record the agent identity and version, and MUST follow the layer's required content, which SHOULD focus on checkable matters: whether statistics are consistent, citations support the claims they are cited for, code runs and reproduces reported numbers, and data match the description.
3. Agent reviews feed only the AI rating. They MUST NOT count toward the expert or community ratings, toward any person's expertise, or toward admission, challenges or appeals.
4. People MAY rate agent reviews for usefulness (section 10); those ratings build the agent's reputation (OIP-9 section 6a).
5. Owners MAY reply to an agent review like any other. Agent reviews MAY be challenged; one that is wrong in a checkable way counts against the agent and its operator (OIP-10 section 1b).

### 7b. Automated verification

1. Where a community enables it, an automated verification layer runs pinned tools on an idea's content: for example checking a formal proof in a proof assistant such as Lean, re-running code against its data and comparing outputs with the reported numbers, or checking data files against their hashes.
2. Each tool and version MUST be pinned in the community setup. Each run MUST be recorded as an input (OIP-13 section 6) with the tool and version, the content hashes checked, the result (passed, failed or could not run) and a log.
3. Results MUST be shown next to the ratings as evidence, not combined into any rating. A community MAY also make a passing check an admission criterion for some subtypes (OIP-11 section 4), for example "the formal proof checks" for `paper/formal-proof`, since that is a checkable fact.
4. **Untrusted code.** Code and data in submissions are untrusted. Every run MUST happen in a fresh, isolated environment with no network access, fixed limits on CPU, memory, disk and time, and no access to key custody (OIP-10 section 2), the content store's write interface, other submissions or any other operator system. A run that exceeds a limit or tries to break isolation MUST be recorded as `could not run`, with the reason.
5. A passing check shows only what the tool checks, for example that a formal statement is proved, not that it matches the informal claim; interfaces MUST say what was checked.

### 8. Review content

1. A review MUST be registered as an OIP-16 idea of type `review` whose `targets` names the reviewed version. An assigned or expert reader review MUST cover: what the work claims, whether the evidence supports the claims, whether the results could be reproduced from the methods, code and data provided, and what should change.
2. Each assigned or expert reader review MUST carry, in the same transaction:
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
4. A community review MUST contain a comment of at least a few sentences, and MAY carry a score and a recommendation on the same scale.
5. A question that does not apply to an idea (for example reproducibility for a `hypothesis`) MAY be answered "not applicable" with a reason. A community MAY add review questions in its setup; it MUST NOT remove the required ones in v1.
6. The owners MAY reply by registering a review of the review.
7. A `revise` recommendation MAY lead the owners to submit a new version, under OIP-11 section 6. A new version starts with no reviews of its own; earlier reviews stay visible in its history.

### 9. Ratings

1. A version's **expert rating** MUST be the weighted median of the scores of its assigned and expert reader reviews, each weighted by the reviewer's weight in the domain (OIP-9 section 6) at the round the review was recorded. Sort scores ascending (ties by reviewer address) and take the first score at which the running total of weights reaches at least half of the total weight.
2. A version's **community rating** MUST be the median of the scores of its community reviews, shown only once at least m_comm community scores exist, and always with their number. Community reviews are not weighted by reputation, so the community rating is shown separately and never mixed into the expert rating.
3. Where the AI agent layer is enabled, the **AI rating** MUST be the median of agent review scores, or a median weighted by agent reputation if the layer says so, and MUST be shown separately and labelled as AI.
4. All ratings MUST be recomputed whenever a review is added or stops counting, and their history MUST be kept.
5. Replications of the version (OIP-16 type `replication`) MUST be shown next to the ratings as counts of each outcome.
6. In v1, ratings MUST NOT change any balance, mint, weight set or α.

### 10. Review ratings

1. Any user with weight greater than zero in the domain MAY rate a review once, with a whole number from 0 (useless) to 10 (careful and very useful), subject to section 4.3. The rating judges the review's care and usefulness, not agreement with it.
2. OIP-9's `quality(r)` is the mean of these ratings, excluding ratings by owners of the reviewed idea (OIP-9 section 4).

### 11. Parameters

| Parameter | Meaning | v1 value | Set by |
| --- | --- | --- | --- |
| k | Assigned reviewers per admitted idea | 3 (a community MAY lower it to 2 if its reviewer pool is too small; the setting is public) | Community |
| f_rev | Payment per assigned review, from the OSO fund | TBD OSO | Community |
| Minimum reviewer weight | Weight needed to review in a domain | TBD | Community |
| Acceptance window | Blocks to accept an invitation | TBD | Community |
| Review deadline | Blocks to deliver after accepting | TBD | Community |
| max_open | Accepted reviews a reviewer may have in progress before being skipped | 3 | Community |
| Community review limit | Community reviews per identity per round | TBD | Community |
| m_comm | Community scores needed before the community rating is shown | 5 | Community |
| Selection method | How assigned reviewers are drawn | Weighted random draw (section 5) | Protocol in v1 |

## Rationale

- **A configurable pipeline.** Fields review differently, and AI agents add a new kind of reviewer. Defining review as a pipeline of layer types, each with its own participants, selection, content, pay and rating, lets a community build the process it needs without a new module, while the principles keep every configuration open and fair.
- **Machines check what machines can check.** Formal proofs and reproducible code can be verified mechanically and cheaply. Recording those results as evidence lets reviewers spend their time on what tools cannot judge, while keeping clear what was and was not checked.
- **AI agents as reviewers, kept separate.** Agents can check statistics, citations and code at a scale people cannot, so they are useful reviewers. Their reviews feed their own rating and build their own reputation, so they inform people without displacing human judgement or letting one operator's agents outvote experts.
- **Four default layers, four purposes.** A spam filter keeps the graph clean; assigned review guarantees every admitted idea a careful look; expert readers keep judging the work as it is used and reproduced; and the community adds feedback and interest from people who are not experts in the field. Keeping the expert and community ratings apart lets both count without letting a crowd outvote experts, or experts silence the crowd.
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

Takes over peer review from OIP-11, which keeps only the rules for new versions. Implements the review layers of Proof of Idea: its spam filter (layer 1, in OIP-11), peer review (layers 2 and 3 here) and public review (layer 4 here). It also follows §3.5 of the 2017 white paper and §5 of the Idea Platform whitepaper, with one network-wide process in place of per-community processes for now. Unlike the 2017 white paper, acceptance is not decided by a majority of reviewers.

## Test Cases

### Weighted median

Scores and weights on one version: A 8 (weight 3), B 2 (weight 1), C 6 (weight 2). Sorted: B, C, A. Total weight 6, half is 3; running totals 1, 3. The rating is 6.

### Required cases

- [ ] An admitted idea gets k invitations drawn by section 5, excluding conflicts and reviewers at max_open.
- [ ] The same seed, candidates and weights always give the same draw.
- [ ] Over many draws, a candidate with twice the weight is drawn about twice as often.
- [ ] An assigned review delivered on time is paid f_rev whatever its score.
- [ ] An expert reader review counts toward the expert rating and is not paid.
- [ ] A community review counts only toward the community rating, which is hidden until m_comm scores exist.
- [ ] A user without expert weight can write a community review but not an assigned or expert reader review.
- [ ] A review registers without validation or stake, and a review found to be spam stops counting.
- [ ] An agent review counts only toward the AI rating and agent reputation, never toward the expert or community rating.
- [ ] With the AI agent layer off (the default), agents cannot review.
- [ ] A headline rating, if configured, shows its weights, and the separate ratings remain visible.
- [ ] An automated verification run records the tool, version, content hashes and result, and is shown as evidence, not as a rating.
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
| Many agents from one operator flooding reviews | Agent reviews feed only the AI rating; per-operator limits; agent reputation starts at zero per version; suspension by the community |
| A crowd brigading an idea through community reviews | Community rating kept separate from the expert rating, shown with its count, never affecting value or expertise; per-identity limits; spam challenges |
| Authors punishing critical reviewers | Pay independent of the verdict; owners' ratings of reviews excluded from quality(r) |
| Submitted code attacking the operator's systems during automated verification | Fresh isolated environment per run, no network, resource limits, no access to custodied keys or other systems (section 7b) |
| Draining the OSO fund | k paid reviews per admitted idea only; admission requires a stake and validation |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
