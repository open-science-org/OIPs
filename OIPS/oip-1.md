---
oip: 1
title: IPFS integration and platform UI
description: Proposes IPFS for storing research artifacts, with IPFS addresses recorded on Ethereum, and asks how researchers should access the platform.
author: Keith Smith (@KeithSSmith)
discussions-to: https://github.com/open-science-org/OIPs/issues/2
status: Stagnant
type: Informational
created: 2017-10-19
---

## Abstract

Legacy OIP, originally written as GitHub issue #2. It proposes storing papers, reviews and data on IPFS and recording only the permanent IPFS address on the Ethereum blockchain, compares three ways of connecting IPFS to the platform (oracles, JavaScript in the web application, or a forked Go IPFS daemon as OpenBazaar did), and asks whether researchers should reach OSO through a website, a standalone application, or both.

## Specification

None. This legacy OIP records an idea and a discussion, not a specification. The original text and discussion are in [issue #2](https://github.com/open-science-org/OIPs/issues/2).

## Rationale

The issue argued that storing full documents on Ethereum is impractical because of gas costs, while an IPFS address stored on-chain gives an immutable, citable reference. It suggested that handling IPFS in JavaScript inside the web application was likely simpler than oracles.

**Later developments.** The 2019 Idea-Hub proof of concept used in-browser IPFS, and a 2020 redesign proposed torrents instead. The v1 design stores content by hash and link and defers decentralized storage to a replaceable Storage module.

## Security Considerations

The issue did not discuss security. Content on IPFS stays available only while some node pins it.

## Copyright

This summary: copyright and related rights waived via [CC0](../LICENSE.md). The original issue text remains with its authors.
