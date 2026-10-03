---
oip: 0
title: OIP purpose and guidelines
description: How OSO Idea Proposals are written, numbered, reviewed and finalized.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Living
type: Meta
created: 2026-10-02
---

## Abstract

An OSO Idea Proposal (OIP) is a design document for the Open Science Organization (OSO) community. It describes a protocol rule, a module, an interface or a process, with a concise technical specification and the reasoning behind it. OIPs are the primary way to propose changes to OSO, collect community input, and record design decisions. This OIP defines the OIP types, statuses, file format and workflow. It is adapted from Ethereum's EIP-1.

## Motivation

From 2017 to 2019, OSO proposals lived as GitHub issues (OIP-1 to OIP-7). Issues are good for discussion but poor as specifications: their text changes without review, they have no status, and there is no single document that implementers can follow. File-based OIPs, reviewed through pull requests, give every decision a stable text, a status, and a reviewable history.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### OIP types

- **Standards Track** — a change that implementations of OSO must agree on. It has one of these categories:
  - **Core** — rules every implementation of the OSO ledger and state machine MUST follow: the idea format, transactions, tokens, minting, value flow, identity records.
  - **Module** — the interface of a replaceable module (validation, review, reputation, edge weights, storage and others) and its reference implementations. Communities choose which module implementations they use.
  - **Interface** — APIs, client conventions and data formats, such as idea metadata or the query API.
- **Meta** — a process around OSO, such as this OIP, governance procedures or the decision-making process. Meta OIPs are binding on the process they describe.
- **Informational** — design notes, guidelines or research that do not require adoption.

### Statuses

| Status | Meaning |
| --- | --- |
| Idea | Not yet tracked; discussed informally before a draft. |
| Draft | The first tracked stage. The OIP is merged into the repository and may change substantially. |
| Review | The authors consider the OIP ready and request peer review. |
| Last Call | The final review window, normally 14 days, before Final. The header gives a `last-call-deadline`. |
| Final | The OIP is settled. Only errata and clarifications may change it; a substantive change needs a new OIP. |
| Stagnant | A Draft, Review or Last Call OIP with no activity for 6 months. Its authors or editors MAY move it back to Draft. Legacy OIPs (see below) are the exception. |
| Withdrawn | The authors have withdrawn the OIP. The number is not reused, and the OIP cannot be revived; a new OIP is needed. |
| Living | Updated continually and never final, such as this OIP. |

A Standards Track OIP MUST NOT move to Last Call while its Security Considerations are incomplete, and a Core OIP MUST NOT move to Final without Test Cases.

### Adoption

Final status means the text is settled. Whether a Final OIP is adopted into the OSO protocol is decided by OSO governance as defined at the current milestone of the OSO roadmap: by the founding and core team before M5, after public advisory votes from M3 onward; by binding community votes from M5; and by full community governance from M6. A later Meta OIP will define those procedures.

### File format

1. An OIP is a Markdown file at `OIPS/oip-N.md`, where N is its number. New drafts use [oip-template.md](../oip-template.md).
2. The file starts with a preamble in YAML front matter, with these fields in this order:

   | Field | Required | Format |
   | --- | --- | --- |
   | `oip` | Yes | The number, assigned by an editor |
   | `title` | Yes | A few words, at most 44 characters, without the OIP number |
   | `description` | Yes | One short sentence, at most 140 characters |
   | `author` | Yes | `Name (@github)` or `Name <email>`, comma-separated |
   | `discussions-to` | Yes | URL of the discussion thread or pull request |
   | `status` | Yes | One of the statuses above |
   | `last-call-deadline` | Last Call only | `yyyy-mm-dd` |
   | `type` | Yes | Standards Track, Meta or Informational |
   | `category` | Standards Track only | Core, Module or Interface |
   | `created` | Yes | `yyyy-mm-dd` |
   | `requires` | If needed | OIP numbers the Specification depends on |
   | `withdrawal-reason` | Withdrawn only | One sentence |

3. The body has these sections in this order. Required sections MUST be present.

   | Section | Required |
   | --- | --- |
   | Abstract | Yes |
   | Motivation | Optional |
   | Specification | Yes |
   | Rationale | Yes |
   | Backwards Compatibility | Optional; required if earlier behaviour changes |
   | Test Cases | Required for Core; optional otherwise |
   | Reference Implementation | Optional |
   | Security Considerations | Yes |
   | Copyright | Yes |

4. Images, diagrams and auxiliary files go in `assets/oip-N/` and are linked relatively, for example `../assets/oip-8/waterfall.svg`. SVG is preferred, then PNG.
5. Other OIPs are linked relatively, for example `./oip-8.md`.
6. Links to external resources SHOULD be limited to permanent references: OSO repositories at a specific commit, published standards (IETF RFCs, W3C recommendations, Ethereum EIPs), and archived papers. Anything else that matters SHOULD be saved as a PDF in the OIP's assets folder.
7. Every OIP MUST end with: `Copyright and related rights waived via [CC0](../LICENSE.md).`

### Workflow

1. **Idea.** Discuss the idea first, in a GitHub issue in this repository or in OSO's community channels, to check whether it is new and wanted.
2. **Draft.** Copy the template, fill it in, and open a pull request with the file named `oip-draft_short_title.md`. An editor checks the format, assigns the next number, and merges it as Draft. Merging a Draft does not mean the OIP is accepted.
3. **Review.** When the authors consider it ready, they open a pull request changing the status to Review and invite reviewers.
4. **Last Call.** After review comments are addressed, the status moves to Last Call with a deadline, normally 14 days later.
5. **Final.** If no substantive change was needed during Last Call, an editor moves the OIP to Final.

Changes to an OIP are always made by pull request. Authors approve changes to their own Draft and Review OIPs.

### Roles

- **Authors** write the OIP, lead its discussion, and build agreement on it. The first author is the champion.
- **Editors** check format and completeness, assign numbers, merge pull requests and update statuses. Editors do not judge the merits of a proposal. An editor MUST NOT be the only editor to act on an OIP they author; until a second editor is added, status changes on the editor's own OIPs need a public approving review on the pull request from someone who is not an author. The current editor is Gajendra Jung Katuwal (@himalayajung); additional editors are added by pull request to this OIP.

### Legacy OIPs

OIP-1 to OIP-7 were written as GitHub issues between 2017 and 2019. Each has a file in `OIPS/` with status Stagnant, a short summary, and a link to its original issue. Their original text and discussion remain in the issues and are not copied, so their authors' wording is not relicensed. Because a legacy OIP has no specification to return to, it is not moved back to Draft; it is revived by opening a new OIP that references it.

## Rationale

EIP-1 is a proven process, used by a large open community for many years, and it already separates the specification (the OIP file) from discussion (the pull request). OSO adds a **Module** category because its design separates a small fixed core from replaceable modules that communities choose. The Adoption subsection is added because OSO's governance changes over the roadmap: the same Final OIP may be adopted by the core team early on and by community vote later. Legacy issue-based OIPs keep their numbers so existing links stay valid.

## Security Considerations

The OIP process is a target for capture: whoever controls merges controls what becomes the specification. Mitigations: all changes go through public pull requests; editors judge format, not merit; adoption into the protocol is a separate governance decision; and the repository history is public, so any change to a Final OIP is visible.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
