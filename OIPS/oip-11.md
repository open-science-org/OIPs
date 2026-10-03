---
oip: 11
title: Submission routing
description: Defines the states a new idea passes through, from submission and pre-screening to validation, challenge and peer review.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Module
created: 2026-10-02
requires: 8, 9, 10, 12, 14, 16
---

## Abstract

Every new idea goes through validation before peer review. Validation is an **admission check** against stated criteria (provenance, required fields, duplication, spam, plagiarism, domain fit, and plausible parents and weights), not a scientific verdict. An AI pre-screen reports first; then N validators drawn from a published roster vote, and a majority admits the idea. Admission mints the idea's tokens into escrow ([OIP-8](./oip-8.md)). A challenge window follows, during which anyone can contest the admission with evidence. Owners can appeal a rejection or a removal once. Peer review then runs separately under [OIP-17](./oip-17.md). This OIP is the reference implementation of the Validation module.

## Motivation

Minting on validation makes the admission gate the main defense against spam, including cheap AI-generated papers. The 2018 Proof of Idea described validation in outline but did not define states, deadlines, conflicts, appeals, or what happens when there are too few validators.

### Prior work

- [Proof of Idea v0.0 (2018), §2.1–2.3 and §3](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): submission and validation of ideas by N validators with a consensus threshold, and the first attack vectors (file size, idea flooding, Sybil identities).
- [OIP-4: Validator merit](./oip-4.md), [OIP-5: Custom validation layers](./oip-5.md) and [OIP-7: Publishing as a cascade of TCRs](./oip-7.md): incentives for validators, community rules, and validation as curated lists.
- [idea-hub issue #18 (2019)](https://github.com/open-science-org/idea-hub/issues/18): a first version of validation layer 1, possibly built on token-curated registries.
- [idea-hub pull request #33 (2020)](https://github.com/open-science-org/idea-hub/pull/33): a Solidity contract with an idea state diagram for registration, validation and publication.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. States

| State | Meaning |
| --- | --- |
| Submitted | Signed by all owners, stake locked (OIP-8 section 8) |
| Prescreened | AI pre-screen report recorded |
| Waiting | Paused without relaxing any rule: no AI budget or permitted model for the pre-screen (OIP-15), too few eligible validators, or the OSO fund cannot pay validator fees (OIP-8 section 4a) |
| InValidation | Validators assigned; votes being collected |
| Returned | Not admitted; returned to the owners with reasons; stake returned once the appeal window closes |
| RejectedSpam | Not admitted as spam; stake slashed once the appeal window closes |
| InAppeal | An appeal of a Returned, RejectedSpam or Overturned decision is being decided |
| Admitted | Admitted to the graph; mint and stake held; challenge window running |
| Challenged | A challenge is being decided; challenge window paused; escrow frozen |
| Overturned | A challenge was upheld; escrow frozen during the appeal window |
| Published | Challenge window closed; escrow and stake released |
| Removed | Admission finally overturned for spam or plagiarism; mint burned; stake slashed |
| Retracted | A Published idea later found to be plagiarism (section 5.7) |
| Withdrawn | Withdrawn by the owners before admission; stake returned |

Allowed transitions:

```
Submitted → Prescreened → InValidation → Admitted → Published → Retracted
Submitted → Waiting → Prescreened                  (pre-screen not yet possible)
Prescreened → Waiting → InValidation               (too few validators, or no fee funds)
InValidation → Waiting → InValidation              (replacements exhausted; cast votes kept)
InValidation → Returned | RejectedSpam
Returned | RejectedSpam → InAppeal → InValidation  (appeal: a fresh draw votes again)
Admitted → Challenged → Admitted | Overturned
Overturned → InAppeal → Admitted | Removed
Overturned → Removed                               (appeal window closes with no appeal)
Submitted | Prescreened | Waiting → Withdrawn
```

Transitions caused by elapsed blocks (deadlines, window closings) run in block-end processing ([OIP-12](./oip-12.md) section 4a). Any other transition MUST be rejected.

### 2. Pre-screen

1. An AI pre-screen MUST record a report: spam likelihood, closest existing ideas with similarity scores, suggested domain, and suggested parents with weights, including uncited dependencies flagged under OIP-14 section 7. The report is recorded as an AI input (OIP-13 section 6) and is a Class A task under [OIP-15](./oip-15.md).
2. The pre-screen MUST NOT reject an idea on its own. It informs validators.

### 3. Validator selection

1. Each domain MUST publish a validator roster on the ledger. A user is eligible for the roster when their weight in the domain (OIP-9 section 6) is at least the community's minimum validator weight, plus any further rules in the community setup (OIP-12).
2. A validator MUST be excluded from an idea if they are an owner, a co-owner with any owner within the last C blocks, an owner of any of the idea's proposed parents, or share an `email-domain` attestation with any owner (OIP-10 section 1).
3. Selection MUST be reproducible: sort the eligible roster by `hash(seed ‖ idea version ID ‖ decision ID ‖ validator address)` and take the first N, where seed is the hash of the block in which the decision opened.
4. In v1 the operator orders transactions and could try to influence the seed. Implementations MUST log the eligible roster and seed with each selection, and MUST NOT describe the procedure as manipulation-resistant.
5. If fewer than the required number of validators are eligible, the idea MUST enter Waiting. Quorum and conflict rules MUST NOT be relaxed.

### 4. Validation vote

1. Each validator MUST vote admit or not-admit before a deadline of V blocks, giving a reason drawn from the stated criteria: provenance, required fields, duplication, spam, plagiarism, domain fit, or implausible parents and weights. A not-admit vote MAY be marked as spam.
2. Validators check the owners' proposed parents and weights against the pre-screen suggestion (OIP-14 sections 6 and 7). Admission approves the proposed weight set.
3. A validator who misses the deadline MUST be replaced by the next eligible validator in the sorted order, and receives the deadline penalty in OIP-9.
4. Each validator has one vote; reputation decides who may serve (section 3.1), not how much a vote counts. More than half of N admit votes moves the idea to Admitted. Otherwise, if more than half of N votes are marked spam, it moves to RejectedSpam; else it moves to Returned.
5. Votes and reasons MUST be public once the vote closes.
6. Validators are paid under OIP-8 section 4a, whatever their vote.

### 5. Challenges, appeals and retraction

1. While an idea is Admitted, any identity MAY file a challenge by locking a challenge stake s_ch and recording evidence of spam or plagiarism. Only one challenge MAY be open at a time. A challenge whose evidence hash matches an already decided challenge on the same idea MUST be rejected.
2. A challenge MUST be decided by a fresh draw of N_ch validators, using the selection rules in section 3 and excluding validators who have already decided on this idea. The challenge window does not run while the idea is Challenged; it resumes when the idea returns to Admitted.
3. A majority upholding the challenge moves the idea to Overturned. If no appeal is filed within A blocks, or the appeal fails, the idea moves to Removed: the escrowed mint is burned and the submission stake slashed (OIP-8). The challenger's stake is returned, and the challenger receives a share r_ch of the slashed submission stake. Validators who voted to admit receive the overturn penalty in OIP-9.
4. If the challenge is not upheld, the idea returns to Admitted and the challenger's stake goes to the OSO fund.
5. The owners MAY appeal once, within A blocks, against Returned, RejectedSpam or Overturned. An appeal is decided by a fresh draw of N_ch validators, excluding validators who have already decided on this idea.
   - An appeal against Returned or RejectedSpam re-runs the validation vote (section 4) with the fresh draw. If it admits the idea, the original not-admit voters receive the overturn penalty in OIP-9.
   - An appeal against Overturned that succeeds returns the idea to Admitted, with the remaining challenge window; the challenge validators who upheld it receive the overturn penalty.
6. A second appeal on the same idea version MUST be rejected.
7. **After publication.** A plagiarism challenge MAY also be filed against a Published idea, decided as in items 2 and 5. If upheld and not reversed on appeal, the idea moves to Retracted: it stays visible and marked as retracted; it MUST NOT be a parent in any new weight set; incoming value it would retain MUST instead be passed to its parents as if α were 0; and the owners receive the misconduct penalty in OIP-9. Value released before retraction is not clawed back in v1.

### 6. Peer review and new versions

1. Peer review is specified in [OIP-17](./oip-17.md). It starts when an idea is Admitted and continues after it is Published.
2. A `revise` recommendation (OIP-17 section 8) MAY lead the owners to submit a new version ([OIP-16](./oip-16.md) section 8). A new version MUST pass the pre-screen. It MUST go through validation again if any of these holds:
   - the pre-screen reports that its content differs from the **last validated** version by more than the community's revision threshold (so small changes cannot add up unchecked);
   - the owners change its parents or weights;
   - one validator, drawn as in section 3, does not confirm within V blocks that the change is minor.

   Otherwise the new version keeps the last validated version's approved weight set.

### 7. Parameters

| Parameter | Meaning | v1 value |
| --- | --- | --- |
| N | Validators per idea | 5 |
| N_ch | Validators per challenge or appeal | 5 (2N+1 recommended once rosters are large enough) |
| V | Vote deadline | TBD blocks |
| C | Co-ownership conflict window | TBD blocks |
| A | Appeal window | TBD blocks |
| s_ch | Challenge stake | TBD OSO |
| r_ch | Challenger's share of a slashed stake | 5,000 bps |
| W | Challenge window (OIP-8) | TBD blocks |
| Minimum validator weight | Weight needed to join a roster | TBD |
| Revision threshold | Content difference that triggers re-validation | TBD |

All parameters are community parameters (OIP-12).

## Rationale

- **Admission, not judgment.** Following the 2026 technical review and OIP-4, validators check stated criteria; scientific disagreement belongs to review, which never sanctions dissent.
- **Pay and sanctions in both directions.** Validators are paid whatever they vote, and wrongful rejections can be overturned on appeal just as wrongful admissions can be overturned on challenge, so neither admitting nor rejecting is the safe choice.
- **Waiting instead of relaxing rules.** Small communities should not silently lower conflict standards.
- **Fresh validators for challenges and appeals.** The original validators have an interest in their own decision. A challenge plus an appeal needs N + 2 × N_ch conflict-free validators; N_ch starts equal to N so that a pilot community can run the process.
- **Burn after appeal.** Burning before the appeal is decided would make a successful appeal impossible to honour.
- **Rewarding challengers.** Without a reward, policing costs a challenger time and risks the stake, with no gain.
- **Re-validation measured from the last validated version.** It keeps revision cheap while stopping a series of small revisions from adding up to different content.

### Open questions

1. Should the challenge window differ by community size?
2. How should the revision threshold be measured?
3. When should N_ch move to 2N+1?

## Backwards Compatibility

Replaces the validation algorithm of Proof of Idea v0.0 (§2.2–2.3), and the v1 design doc's routing description.

## Test Cases

To be added. Required cases: each disallowed transition is rejected; an idea with too few eligible validators stays Waiting; the same seed and roster always select the same validators; a challenge pauses the window; a successful challenge with no appeal burns the whole mint and slashes the stake; a successful appeal after a challenge restores the idea with nothing burned; a repeated challenge with the same evidence is rejected; a second appeal is rejected; a minority not-admit vote on an upheld admission carries no penalty; a not-admit vote overturned on appeal carries the overturn penalty.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Spam submissions | Stake, pre-screen, admission vote, challenge, burn and slash |
| Biased validator draw by the operator | Logged roster and seed, disclosed trust assumption, later randomness design (OIP-13) |
| Collusion between authors and validators | Conflict rules, random draw, fresh validators for challenges and appeals |
| Validators biased toward admitting or rejecting | Fee paid whatever the vote; penalties for decisions overturned in either direction |
| Frivolous challenges | Challenge stake forfeited on failure; one open challenge at a time; repeated evidence rejected |
| Griefing by repeated challenges to freeze escrow | Window paused only while a challenge is open; each failed challenge costs s_ch |
| Censorship of unpopular work | Admission criteria exclude scientific merit; Returned ideas get reasons and one appeal |
| Plagiarism found after the window | Retraction stops further retained income and redirects it to the parents |
| Small revisions adding up to new content | Difference measured from the last validated version; one validator confirms minor changes |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
