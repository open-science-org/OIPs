---
oip: 14
title: Idea attribution and value flow
description: Defines how ideas are linked by intrinsic and extrinsic evidence over time, how links are approved, and how value flows along them.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Core
created: 2026-10-02
requires: 8, 10, 11, 12, 13
---

## Abstract

Ideas are connected by two forces. **Intrinsic** links come from the content itself: how much one idea actually depends on another, as assessed by an AI model reading both. **Extrinsic** links come from the environment: citations, the parents authors declare, and confirmations by validators and reviewers over time. A third factor, **time**, sets the direction: credit flows only from a later idea to an earlier one.

This OIP defines the knowledge graph and the narrower payout graph. It specifies how intrinsic assessments and extrinsic evidence are recorded and combined into suggested weights, and how a link becomes a payout edge only through explicit approval. It also specifies the payout waterfall that moves value along approved edges, with exact integer accounting, dust handling and bounded settlement. Uncited dependencies found by AI are surfaced, but they never move value unless approved.

## Motivation

The 2018 Generalized Idea Protocol (GIP) described a graph of ideas with weighted edges for idea flow and value flow, and defined weights as normalized mutual information. In practice, weights were left to "the market". Two sources of evidence are now available, and they disagree in useful ways:

- **Citations** record credit as people chose to give it. They are shaped by habit, prestige and omission.
- **Content analysis** by language models can estimate actual dependence, including dependencies that were never cited. It cannot tell who learned from whom.

Time resolves part of the ambiguity: a later idea can depend on an earlier one, never the reverse. The 2026 technical review also required that value move only along approved, acyclic links, with exact accounting. This OIP brings those pieces together. It takes over the value-flow rules first drafted in [OIP-8](./oip-8.md), which now covers tokens only.

### Prior work

- [Generalized Idea Protocol (GIP, 2017–2018)](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/README.md): ideas as a growing graph, idea flow and value flow, and GIP v0.0 with equal weights that authors and reviewers could redistribute.
- [OSO: An Idea Platform v0.3 (2018), §4.2](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): weights defined as normalized mutual information, the absorption coefficient α, and value flow-back to parents.
- [OSO white paper (2017), §3.6.2](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_white_paper.pdf): the cost of idea.
- [Proof of Idea v0.0 (2018), §2.5](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): 15% of each mint to the cited publications.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Terminology

| Term | Meaning |
| --- | --- |
| Knowledge graph | All recorded links between ideas, of any type |
| Payout graph | The subset of links approved as payout edges, with approved weights |
| Origin date | The calendar day an idea version first existed: its publication date for imported works, or the day label of its registration block for new submissions (OIP-13 section 2) |
| Intrinsic assessment | A recorded AI judgment of how much a child version depends on a candidate parent, from content alone |
| Extrinsic evidence | Declared citations, author-proposed parents and weights, and recorded confirmations or disputes by validators and reviewers |
| Candidate edge | A possible payout edge from a child version to a parent version, before approval |
| Weight set | The approved parents and weights of one child version, effective from a stated block |

Amounts, weights, α and scores are integers. Weights and scores are in bps (10,000 = 1). Base units are as defined in OIP-8.

### 2. Link types

The knowledge graph MUST support these link types:

| Type | Meaning | Can be a payout edge |
| --- | --- | --- |
| `depends-on` | The child builds on the parent | Yes, once approved |
| `cites` | A declared citation, recorded as extrinsic evidence | Only as part of an approved `depends-on` edge |
| `similar` | Related content without an established dependency | No |
| `contradicts` | The child disputes the parent | No |
| `reviews` | The child is a review of the parent (OIP-17) | No |
| `replicates` | The child replicates the parent's result | No, until a later OIP specifies otherwise |
| `version-of` | The child is a later version of the parent | No |

### 3. Time

1. Every idea version MUST have an origin date recorded at registration, as a calendar day `yyyy-mm-dd`. For imported works it is the source's publication date, recorded as an input (OIP-13 section 6). For new submissions it is the day label of the block in which the version was registered (OIP-13 section 2). Both are recorded values: the state machine compares them and never reads a clock.
2. A payout edge from child i to parent j MUST satisfy one of:
   - `origin_date(j) < origin_date(i)`; or
   - both are new submissions with the same origin date, and j was registered in an earlier block than i.

   Any other edge MUST NOT be approved as a payout edge; it MAY be recorded as `similar`.
3. Because every payout edge points strictly back in time, the payout graph is acyclic. Implementations MUST still reject self-edges and cycles explicitly.
4. A strong intrinsic link between ideas with close origin dates SHOULD be presented as possible independent discovery, not as dependence.

### 4. Intrinsic assessments

1. An intrinsic assessment MUST be recorded as an AI input transaction (OIP-13 section 6) containing: the child version, the candidate parent version, a dependence score in bps, a rubric category, references to the passages used as evidence, and the model and configuration identifiers.
2. The rubric categories are: `essential-method`, `data-dependency`, `direct-extension`, `background`, `critique`, `none`. A community MAY add categories suited to its field in its setup (for example `uses-code` or `legal-precedent`); added categories MUST map to one of these for any rule that depends on the category.
3. An assessment MUST NOT change any balance or weight by itself.
4. A newer assessment of the same pair MAY be recorded, for example by a newer model. It does not replace the old record, and it does not change an approved weight set unless that set is re-approved (section 7).
5. Assessments that can lead to payout edges are Class A tasks under [OIP-15](./oip-15.md) and require two independent models.

### 5. Extrinsic evidence

1. A declared citation from the child to the parent MUST be recorded as a `cites` link, from the submission or from imported metadata.
2. The child's owners MAY propose parents and weights in their signed submission. Proposed weights MUST sum to 10,000.
3. Validators and reviewers MAY record a confirmation or a dispute of a specific edge, with a reason. These records accumulate over time.
4. The extrinsic score of a candidate edge is:
   - the owners' proposed weight, if they proposed one;
   - otherwise, if the child cites the parent, 10,000 divided by the number of the child's declared citations to works in the graph, rounded down;
   - otherwise 0.

### 6. Suggested weights

For each candidate parent j of child i, the suggested raw score is:

```
s_ij = floor( ( λ × intrinsic_ij + (10,000 − λ) × extrinsic_ij ) / 10,000 )
```

where λ is a community parameter in bps (OIP-12). The suggested weights are the raw scores of the candidates normalized to sum to 10,000. Normalization MUST use floor division, with the residue added to the candidate with the highest raw score and, for ties, the lowest work ID. Candidates with a raw score of 0 are dropped.

Suggested weights are a proposal for approvers. They MUST NOT be used for payouts until approved.

### 7. Approval

1. A payout edge and its weight become effective only through an **approval** transaction that records a complete weight set for one child version, the approving authority, and the block from which it takes effect.
2. Approving authorities:

   | Child | Approved by |
   | --- | --- |
   | New submission | Its admission (OIP-11): plausible parents and weights are an admission criterion, validators check the owners' proposed weights against the suggestion, and admission approves them |
   | Imported work in the curated slice | The disclosed weight panel of the community |
   | Imported work outside the curated slice | No one; it has no payout edges |
   | Re-approval for a new submission | A fresh draw of N validators under OIP-11 section 3, excluding conflicts |
   | Re-approval for a curated import | The disclosed weight panel |

3. **Uncited dependencies.** A candidate with a strong intrinsic score (at least the community's flag threshold) and no extrinsic evidence MUST be flagged. A flagged candidate:
   - MUST be shown to validators during admission and to the child's owners;
   - MAY be added to the weight set only if the child's owners accept it, or if the approving authority approves it after giving the owners a chance to respond;
   - MUST NOT move value until it is part of an approved weight set.
4. A new weight set MUST apply only to payments made from its effective block onward. It MUST NOT change past payouts.
5. A dispute of an approved edge MAY lead the approving authority to re-approve the weight set.
6. A candidate edge to an idea that shares an owner with the child (a self-citation) MUST be flagged to the approving authority, which MAY cap or reject its weight.

### 8. Payout waterfall

Each idea I keeps, for its current weight set, three cumulative counters: `num_I` (the sum of `v × (10,000 − α_I)` over incoming amounts), `up_I` (the total due upstream) and `paid_Ij` (the total already transferred to each parent j). When an amount v arrives at idea I, whose retention is α_I and whose approved weight set has parents j with weights w_j, the ledger MUST compute:

```
num_I      = num_I + v × (10,000 − α_I)
upstream   = floor( num_I / 10,000 ) − up_I          (this payment's share for parents)
up_I       = up_I + upstream
retained   = v − upstream
owed_Ij    = floor( up_I × w_j / 10,000 ) − paid_Ij  for each parent j
```

1. `upstream` MUST stay in I's idea account, held for parents, and MUST NOT be distributed to holders. `owed_Ij` is a pending liability from I to j; it is transferred under section 10, after which `paid_Ij` increases by the amount transferred, and the transferred amount becomes incoming value at j, where the waterfall repeats.
2. `retained` MUST be distributed to I's IDEA holders pro rata to their holdings at that block (OIP-8 section 9), with floor rounding. The distribution residue MUST stay in I's idea account and be added to its next distribution.
3. Because the counters are cumulative, the totals due to parents depend only on the total amount received, not on how it was split into payments. Held amounts never exceed what parents are owed plus fewer than one base unit per parent.
4. When a new weight set takes effect, the amount held for parents but not yet owed under the old set (`up_I − Σ floor(up_I × w_j / 10,000)`) MUST be carried into the new set as its opening `up_I`, with `num_I = up_I × 10,000` and `paid_Ij = 0`; amounts already owed under the old set MUST still be transferred to the old parents.
5. An idea with no approved weight set MUST retain all incoming value. A Retracted idea (OIP-11 section 5.7) MUST be treated as having α = 0 from the block of its retraction.
6. IDEA units held by a claim-slot or unresolved-claims account MUST accrue to that account and remain an identifiable liability until claimed (OIP-10).
7. α MUST be set by the community the idea belongs to. Authors MUST NOT set their own α.

Income types:

| Income type | Enters the waterfall |
| --- | --- |
| Cited share of a child's mint (OIP-8 section 4), split among the child's parents by OIP-8 | Yes, at each parent |
| Funding sent to an idea | Yes |
| Gifts to an idea | Yes |
| Value passed up from a child | Yes |
| Review bounties and validator pay | No; these are payments to people for tasks |
| Restricted grants for a stated purpose | No, until a later OIP specifies otherwise |

### 9. Traversal

1. A shared ancestor reached through several branches MUST receive each branch's amount separately.
2. Traversal MUST be breadth-first from the paying idea, with each idea's parents processed in ascending work-ID order.
3. A cited work that is not in the graph has no payout edge and takes no part in the weights (section 5). No share is reserved for it, and nothing is paid retroactively if it is imported later.
4. Settlement work MUST be processed from a queue in the order it was created. Each block MUST process at most the per-block settlement cap of queue entries; the rest stays queued, as pending liabilities, for the next block.

### 10. Dust and pending liabilities

1. When `owed_Ij` (section 8) reaches the dust threshold D base units, a transfer for the pair (I, j) MUST be queued (section 9), unless one is already queued. When it runs, it transfers the whole `owed_Ij` at that moment.
2. In the last block of every round of K blocks, in block-end processing (OIP-12 section 4a), a transfer MUST be queued for every pair with `owed_Ij` greater than zero and none already queued, in ascending order of I's work ID and then j's work ID.
3. Amounts that these transfers create further upstream follow the same rules: they are queued at once if they reach D, and otherwise wait for the next round's end. This guarantees that each round's settlement terminates.

### 11. Parameters

| Parameter | Symbol | v1 value | Set by |
| --- | --- | --- | --- |
| Intrinsic share of suggested weights | λ | 5,000 bps | Community |
| Flag threshold for uncited dependencies | — | TBD bps | Community |
| Retention | α | 8,000 bps (default) | Community |
| Dust threshold | D | TBD base units | Protocol |
| Settlement round | K | TBD blocks | Protocol |
| Per-block settlement cap | — | Set from measurements of the seed graph | Protocol |

## Rationale

### Two forces, kept separate

Recording intrinsic and extrinsic evidence separately, instead of storing one blended number, keeps disagreement between them visible:

| Intrinsic | Extrinsic | Likely meaning |
| --- | --- | --- |
| Strong | Strong | Real, acknowledged dependency |
| Strong | None | Uncited dependency, rediscovery, or possible plagiarism |
| Weak | Strong | Background, courtesy or prestige citation |
| Weak | Weak | No real relation |

### Approval before money moves

AI assessments are judgments that can be wrong, differ between models and be manipulated by adversarial text. An AI estimate alone therefore never moves value. Uncited dependencies are surfaced to the people who can act on them, which addresses missing credit without making a model the arbiter of who owes whom.

### Time as the direction of credit

Requiring parents to be strictly older gives acyclicity for free, matches how ideas actually build on each other, and lets close origin dates be read as possible independent discovery instead of copying.

### λ per community

Fields differ in citation practice. A field with sparse or prestige-driven citation can rely more on intrinsic evidence; a field with careful citation can rely more on extrinsic evidence. This follows the modular design of OIP-12.

### Weights fixed until re-approved

Extrinsic evidence keeps arriving for years, but payouts must be predictable and replayable. New evidence leads to a new approved weight set that applies forward, never to recalculated history.

### Further research

Estimating how much one idea owes to another is an unsolved research problem. This OIP makes attribution safe to operate (approval before value moves, recorded evidence, weights applied only forward), but it does not make the weights accurate. Further research is needed in at least these areas:

1. **Ground truth.** Build a benchmark of child–parent pairs whose dependence has been rated by several domain experts, and measure how much experts agree with each other. Model quality cannot be judged without knowing how consistent human judgment is.
2. **Citation function.** Classify why a work is cited (method used, data used, extended, compared, criticized, background), building on existing research into citation intent, and test how well the rubric categories in section 4 match expert judgment.
3. **Model reliability.** Measure how stable intrinsic scores are across models, model versions and prompt wording, and how well calibrated the scores are against the benchmark.
4. **Robustness.** Test how easily scores can be manipulated by adversarial wording, strategic citation and omitted citations, and develop defenses.
5. **Beyond similarity.** Explore measures closer to the 2018 idea of mutual information, such as whether a child's key results could be produced without a given parent, and methods analogous to training-data attribution in machine learning.
6. **Time.** Distinguish dependence from independent discovery, handle preprints and later versions, and decide how credit should treat ideas that were overlooked and rediscovered.
7. **Other kinds of ideas.** Extend attribution to code, datasets, reviews and replications, where citation conventions are weaker.
8. **Choosing λ.** Find out, field by field, how much weight intrinsic evidence should get relative to extrinsic evidence.

Results from this research SHOULD lead to revisions of this OIP, of the rubric, and of community parameters such as λ and the flag threshold.

### Open questions

1. Value of the flag threshold, and whether it should depend on the rubric category.
2. Should `replicates` links ever move value, for example to reward replications?
3. How should origin dates of preprints and later journal versions be handled?
4. Should a `critique` dependency carry a lower weight by default?

## Backwards Compatibility

- Replaces the edge definition of the GIP repository (v0.0, equal weights adjustable by the author) and §4.2 of the OSO Idea Platform whitepaper v0.3, which defined weights as normalized mutual information.
- Takes over the payout waterfall, payout-graph rules and dust rules from earlier drafts of OIP-8 (its former sections 5–7), with one change: the explicit no-cycles rule is now a consequence of the time rule, and is still checked.

## Test Cases

All amounts are in whole OSO for readability; implementations use base units.

### Payout waterfall

Idea I receives 100 OSO of funding. α = 8,000 bps. Approved parents: P1 (w = 6,000), P2 (w = 4,000). IDEA holdings: A 600,000, B 250,000, investor 150,000.

| Step | Amount |
| --- | --- |
| upstream = 100 × 2,000 / 10,000 | 20 |
| to P1 = 20 × 6,000 / 10,000 | 12 |
| to P2 = 20 × 4,000 / 10,000 | 8 |
| retained = 100 − 20 | 80 |
| A (60%) | 48 |
| B (25%) | 20 |
| investor (15%) | 12 |
| **Total** | **100** |

### Split payments

D = 2,000 base units, α = 8,000 bps, one parent with w = 10,000.

- One payment of 10,000 base units: upstream 2,000 ≥ D, settled at once. The parent receives 2,000.
- Ten payments of 1,000 base units in one round: each adds 200 to `up_I`; `owed_Ij` reaches 2,000 after the tenth payment and settles. The parent receives 2,000.
- Uneven amounts: one payment of 9,990, or ten payments of 999, both give `up_I = floor(19,980,000 / 10,000) = 1,998`. By the end of the round the parent has received 1,998 in both cases.

### Suggested weights

λ = 5,000. Child C cites A and B and proposes no weights. Intrinsic scores: A 9,000, B 1,000. Extrinsic scores: A 5,000, B 5,000 (two citations).

- s_CA = (5,000 × 9,000 + 5,000 × 5,000) / 10,000 = 7,000
- s_CB = (5,000 × 1,000 + 5,000 × 5,000) / 10,000 = 3,000
- Suggested weights: A 7,000, B 3,000. No value moves until a weight set is approved.

### Required cases

- [ ] An edge to a parent with the same or a later origin date is rejected as a payout edge.
- [ ] A flagged uncited dependency moves no value before approval, and moves value only from its approval block afterwards.
- [ ] A re-approved weight set changes later payouts and leaves earlier payouts unchanged.
- [ ] A newer intrinsic assessment alone changes no weight and no balance.
- [ ] Every waterfall satisfies debits = credits + escrow + pending liabilities.
- [ ] The split-payment case gives identical end-of-round balances for one and ten payments.
- [ ] Two implementations replaying the same log give identical balances.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Adversarial text written to inflate intrinsic scores | Assessments never move value alone; approval required; assessments recorded with evidence for audit |
| Citation stuffing to route value to one's own ideas | Extrinsic score split across all declared citations; validators compare proposed weights to the suggestion; self-citations flagged to the approving authority (section 7) |
| Hiding a dependency by not citing it | Intrinsic flagging during admission; challenge for plagiarism (OIP-11) |
| Back-dating an imported work to become a parent | Origin dates of imports recorded from the source with the import; disputes go to the approving authority |
| Registering a copy of someone's unpublished or not-yet-imported work first, to become its parent | Plagiarism checks at admission, challenge during the window, and retraction after publication (OIP-11 section 5) |
| Retroactive rewriting of payouts | Weight sets apply only from their effective block |
| Rounding differences between implementations | Integer arithmetic, floor rounding, fixed residue destinations, deterministic order |
| Unbounded traversal | Per-block settlement cap; excess work queued as pending liabilities |
| Splitting payments to avoid paying parents | Cumulative counters make totals independent of how payments are split; pending liabilities settled per round |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
