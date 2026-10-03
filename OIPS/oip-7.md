---
oip: 7
title: Publishing as a cascade of TCRs
description: Models decentralized publishing as a cascade of information filters, each implemented as a token-curated registry.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: https://github.com/open-science-org/OIPs/issues/9
status: Stagnant
type: Informational
created: 2019-04-09
---

## Abstract

Legacy OIP, originally written as GitHub issue #9. It models publishing as a large information filter that formats, quality-controls and segregates research results, and observes that this filter is a cascade of smaller filters with different uses (for example a list of the best CRISPR posters, or a journal for applied medical-imaging results). Each filter produces a curated list with incentives for its curators, which matches the structure of a token-curated registry (TCR). Decentralized publishing could therefore be built as a cascade of TCRs, and a group of researchers could start a new publishing channel by creating a new TCR.

## Specification

None. The original text is in [issue #9](https://github.com/open-science-org/OIPs/issues/9). The issue was marked work in progress.

## Rationale

The layers of Proof of Idea (spam filter, peer review, public review, sub-networks) already form such a cascade. Modeling each as a TCR would allow one general implementation with different instances for each use. Related: Idea-Hub issue #18 discussed using an existing TCR implementation for the first validation layer.

**Later developments.** The v1 design keeps the cascade idea through communities, channels and replaceable Validation and Review modules, without committing to TCRs.

## Security Considerations

Not discussed in the issue. Token-weighted curation can be captured by large token holders.

## Copyright

This summary: copyright and related rights waived via [CC0](../LICENSE.md). The original issue text remains with its authors.
