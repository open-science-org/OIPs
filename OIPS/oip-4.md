---
oip: 4
title: Basic validation layer and validator merit
description: Asks how to encourage meaningful participation by validators in the Proof of Idea spam-filter layer.
author: Guilherme Gervasio (@gil-air-may)
discussions-to: https://github.com/open-science-org/OIPs/issues/6
status: Stagnant
type: Informational
created: 2019-02-21
---

## Abstract

Legacy OIP, originally written as GitHub issue #6. Assuming validator votes are public, it asks how to reward responsible validators and discourage careless or ill-intentioned ones in the basic validation layer of Proof of Idea, listing cases to avoid: too few validators taking part, validators who vote without reading, approval of inappropriate content, and rejection of good ideas in bad faith.

## Specification

None. The original text and discussion are in [issue #6](https://github.com/open-science-org/OIPs/issues/6).

## Rationale

The discussion described the basic layer as a spam filter, like the vetting step of preprint servers, rather than a judgment of quality. It noted that Proof of Idea already rewards only validators who vote, and suggested keeping that simple binary incentive at first, with a staking-based (linear) incentive considered later.

**Later developments.** The v1 design treats validation as an admission check against stated criteria, pays validators for voting on time regardless of their vote ([OIP-8](./oip-8.md)), and sanctions only admission decisions overturned on challenge, never scientific disagreement.

## Security Considerations

Rewarding agreement with the majority encourages conformity; not rewarding care encourages rubber-stamping.

## Copyright

This summary: copyright and related rights waived via [CC0](../LICENSE.md). The original issue text remains with its authors.
