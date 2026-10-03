---
oip: 16
title: Idea object
description: Defines what an idea is, its stable work ID, its immutable versions, and the registry state kept separately from content.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Core
created: 2026-10-03
requires: 10, 13
---

## Abstract

An **idea** is any intellectual contribution that added new information when it was created: a paper, preprint, dataset, code, review, replication or hypothesis. This OIP defines the idea object that every other OIP refers to. An idea has three layers: a stable **work** that identifies it over time; immutable, content-addressed **versions** that record what the authors claimed at each point; and a mutable **registry record** that holds everything that changes without new work (status, owners, approved weights, claims). It specifies the version fields, how a version ID is computed, which changes create a new version, the idea types, how imported works are represented, and what is deliberately left out of the object.

## Motivation

The Generalized Idea Protocol (GIP, 2017–2018) set out the idea at the center of OSO: research can be described by idea objects, the relationships among them, and the state changes caused by users. The 2018 Idea Platform whitepaper (§4.1) sketched an idea state with an ID equal to the "hash of the state + content", together with name, ownership, wallet, license, relationships, content, reputation and reviews. The GIP repository's `models/Idea.py` added a previous-version pointer (`source`), supporting files, authors, events and keywords.

That sketch mixed two kinds of data. If the ID hashes ownership, status and reviews together with content, then every ownership change or new review makes the idea look like a new piece of work, and nothing can say which content a signature or a payout edge refers to. The 2026 technical review asked for a stable work ID, an immutable ID per version, and registry state kept apart from content. OIPs 8, 10, 11, 12 and 14 already assume this split but no OIP defines it. Implementations need one exact definition so that two nodes compute the same version IDs and accept the same ideas.

### Prior work

- [Generalized Idea Protocol (GIP, 2017–2018)](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/README.md): an idea as "an entity which generates new information at the time of its creation", and research as idea objects, their relationships and their state changes.
- [GIP `models/Idea.py`](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/models/Idea.py), with [`IdeaType.py`](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/models/IdeaType.py), [`Event.py`](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/models/Event.py) and [`Author.py`](https://github.com/open-science-org/GIP/blob/78456634f6d9a0170887fa5b0d01eacd804b5fb6/models/Author.py): the first idea class, with a previous-version pointer, supporting files, authors, events and keywords.
- [OSO: An Idea Platform v0.3 (2018), §3.1 and §4.1](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/OSO_Idea_Platform_whitepaper.pdf): an idea state with ID, name, ownership, wallet, license, relationships, content, reputation and reviews.
- [Technical design v0 (2018)](https://github.com/open-science-org/OSO/blob/898ee42ebeb9ea7248214fa7c508a318df144d5d/OSO_design_v0.pdf): a minimal idea class, versioning and content addressing as open questions, and a suggestion to build on a metadata standard such as Dublin Core.
- [Proof of Idea v0.0 (2018), §6](https://github.com/open-science-org/wiki/blob/52ba175b3bc57a8c08297c4e4a3db835ea2edbde/Proof_of_Idea.pdf): "How to handle version change?" left as a to-do.
- [idea-hub issues #5 (2018) and #24 (2020)](https://github.com/open-science-org/idea-hub/issues/5): idea metadata stored in a distributed store, and later as a `metadata.json` inside each idea's torrent ([#24](https://github.com/open-science-org/idea-hub/issues/24)).
- [OIPs issue #4 (2018)](https://github.com/open-science-org/OIPs/issues/4): renaming the Interplanetary Idea System to the Generalized Idea Protocol.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Terminology

| Term | Meaning |
| --- | --- |
| Idea | An intellectual contribution that added new information when it was created, registered on the OSO ledger. Made of one work, one or more versions, and one registry record. |
| Work | The idea over time. Identified by its work ID, which never changes. |
| Version | An immutable record of the idea's content and claims at one point in time. Identified by its version ID. |
| Registry record | The mutable state of a work, changed only by ledger transactions defined in other OIPs. |
| Content | The files that carry the idea (paper, data, code), referred to by hash and location, not stored by OSO in v1. |
| Registrant | The identity whose transaction registered a version: an owner for new submissions, the ingestion identity for imports (OIP-10 section 3). |

### 2. What is an idea

1. An idea MUST be a contribution that added new information when it was created. Restating, reformatting or translating an existing idea without new content is not a new idea; it is either a new version of the same work (by its owners) or a duplicate.
2. Each idea MUST have one of these types:

   | Type | Meaning | Extra requirement |
   | --- | --- | --- |
   | `paper` | A peer-reviewed or formally published article, book or chapter | — |
   | `preprint` | A publicly posted manuscript not yet formally published | — |
   | `dataset` | A collection of data | At least one content reference to the data |
   | `code` | Software, a model or an analysis pipeline | At least one content reference to a fixed release or commit |
   | `review` | A review of another idea ([OIP-17](./oip-17.md)) | Exactly one `reviews` target version |
   | `replication` | An attempt to reproduce another idea's result | Exactly one `replicates` target version, and the outcome (`reproduced`, `partially`, `not-reproduced`) |
   | `hypothesis` | A stated, testable proposal not yet tested | — |
   | `other` | Anything else that meets item 1 | A one-line description of the kind |

3. These are not ideas and MUST NOT be registered as idea objects: chat messages and AI conversations (OIP-15), votes and ratings, comments without new content, and ledger transactions themselves.
4. A community MAY restrict which types it accepts (OIP-12). It MUST NOT redefine the types.

### 3. Work ID

1. The work ID MUST be the version ID of the work's first version.
2. A work ID MUST NOT change, including when owners, status, license or content change in later versions.
3. Each ledger account that belongs to an idea (OIP-8 section 2) MUST be keyed by work ID.

### 4. Versions

A version MUST be a JSON object with exactly these fields. Fields marked optional MAY be empty (an empty string, list or object, or `null` where shown) but MUST be present.

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `schema` | string | Yes | `oso.idea/1` for this OIP |
| `type` | string | Yes | One of the types in section 2 |
| `title` | string | Yes | At most 300 characters |
| `abstract` | string | Yes | At most 5,000 characters |
| `domain_tags` | list of strings | Optional | Domains or keywords proposed by the authors. The authoritative domain similarity is a recorded AI input that the community can correct (OIP-9, OIP-15) |
| `content_refs` | list of objects | Yes, see section 5 | Hashes and locations of the content |
| `previous` | string or `null` | Yes | Version ID of the version this one replaces; `null` for a first version |
| `parents` | list of objects | Optional | Parents proposed by the owners: `{ "version_id", "weight_bps" }`, weights summing to 10,000 when present (OIP-14 section 5) |
| `cites` | list of objects | Optional | Declared citations: `{ "version_id" }` for works in the graph, or `{ "external_id" }` (for example `doi:10.1000/xyz`) for works outside it |
| `targets` | object | Required for `review` and `replication`; `{}` otherwise | `{ "reviews": version_id }` or `{ "replicates": version_id, "outcome": ... }` |
| `license` | string | Yes | An SPDX license identifier, for example `CC-BY-4.0`, or `LicenseRef-` followed by a URL-safe name for a license defined in the content |
| `ai_disclosure` | object | Yes | `{ "level": "none" | "assisted" | "generated", "description": string }` (OIP-10 section 7) |
| `owners` | list of objects | Yes for new submissions; empty for imports | Owners at registration: `{ "identity", "share_bps" }`, summing to 10,000 (OIP-10 section 4) |
| `attribution` | list of objects | Yes | The people credited, in the order the work lists them: `{ "name", "orcid" }`, with `orcid` empty if unknown |
| `external_ids` | object | Optional; required for imports | Identifiers elsewhere, for example `{ "doi": ..., "arxiv": ..., "openalex": ... }` |
| `registrant` | string | Yes | Address of the registering identity |

1. Extra fields MUST be rejected. A later schema version (`oso.idea/2`) MAY add fields through a new OIP.
2. A version MUST NOT contain status, current owners, IDEA balances, approved weights, reviews received, ratings, reputation, origin dates or any other registry state (section 7).
3. The serialized version MUST NOT exceed 64 KiB.

### 5. Content references

1. Each entry of `content_refs` MUST be `{ "role", "hash", "uri", "media_type" }`, where `role` is `main`, `supplement`, `data` or `code`.
2. `hash` MUST be the SHA-256 digest of the exact bytes of the file, written as `sha256:` followed by 64 lower-case hexadecimal characters. For code, the hash MAY instead identify a commit, written as `git:` followed by the full commit hash, with `uri` pointing to the repository.
3. A new submission MUST include at least one `main` reference with a hash, so that the content the owners signed cannot be swapped later behind the same location.
4. An imported work MAY have references with an empty `hash` when the content is not openly available; its `external_ids` then identify it.
5. OSO does not store content in v1. A reference whose location stops working does not change the version; owners MAY register a new version with a new location and the same hash.

### 6. Version ID

1. The version ID MUST be `sha256:` followed by the lower-case hexadecimal SHA-256 digest of the version serialized with the JSON Canonicalization Scheme (RFC 8785), encoded as UTF-8.
2. Two versions with the same fields MUST have the same ID. Because `registrant` and `owners` are fields, the same content registered by different people gives different version IDs; whether it is a duplicate is decided at admission (OIP-11), not by the ID.
3. A transaction that registers a version MUST carry the version and its ID. The ledger MUST recompute the ID and reject the transaction if they differ, or if a version with that ID already exists.
4. A new submission's registration transaction MUST be signed by every owner listed in `owners` (OIP-10 section 4), over the transaction's typed-data hash, which includes the version ID (OIP-13 section 1).

### 7. Registry record

Each work MUST have one registry record, created when its first version is registered. It holds:

| Entry | Defined in |
| --- | --- |
| List of version IDs in registration order, and the current version | This OIP, section 8 |
| Origin date of each version | OIP-14 section 3 |
| Routing status of each version | OIP-11 section 1 |
| Community the work was submitted to | OIP-12 section 5 |
| Current owners and IDEA units | OIP-8 section 9, OIP-10 section 4 |
| Attribution corrections and claim slots | OIP-10 sections 4 and 6 |
| Approved weight set and its effective block | OIP-14 section 7 |
| Links recorded by others (`cites` from children, `similar`, `contradicts`, confirmations and disputes) | OIP-14 section 2 |
| Retraction | OIP-11 section 5 |

The registry record MUST change only through transactions defined in an OIP. Its changes never alter any version or its ID.

### 8. New versions

1. A new version MUST name the work's current version in `previous`, and MUST be registered by a transaction signed by all current owners. For an imported work, only an adjudicated claimant (OIP-10 section 6) or the ingestion identity correcting import metadata MAY register a new version.
2. A change to any version field (section 4) MUST be made by a new version. This includes the title, abstract, content, parents, citations and license. A new license applies to the new version only; earlier versions keep theirs.
3. A change to registry state MUST NOT create a version.
4. Whether a new version needs validation again is decided by OIP-11 section 6. A new version MUST NOT mint (OIP-8 section 3).
5. Versions form a single chain per work: two versions MUST NOT name the same `previous`.

### 9. Imported works

1. An imported work MUST be registered by the ingestion identity under its delegation (OIP-10 section 3), with `owners` empty, `attribution` and `external_ids` taken from the source, and `registrant` set to the ingestion identity.
2. Its origin date, from the source's publication date, is recorded in the registry (OIP-14 section 3), not in the version.
3. The ledger MUST keep an index of external identifiers and MUST reject a registration whose DOI, arXiv ID or OpenAlex ID already belongs to another work. A preprint and its later journal version MAY be registered as two versions of one work, or as two works linked by a later OIP; until that OIP exists, they are two works.

### 10. Display names

An idea has no on-ledger human-readable name beyond its title. Interfaces SHOULD show the title with a short form of the work ID, for example its first 12 hexadecimal characters.

## Rationale

### Three layers instead of one hash

The 2018 design hashed state and content together. Splitting them gives each consumer what it needs: payout edges and signatures point at exact content (version ID); balances, ownership and accounts follow the idea over time (work ID); and status, owners and weights can change without pretending to be new work (registry). This is how the other v1 OIPs already use ideas.

### Mapping from the 2018 sketch and `Idea.py`

| 2018 field | Here |
| --- | --- |
| ID (hash of state + content) | Version ID (content only) and work ID (stable) |
| Name | `title` |
| Ownership | `owners` at registration; current owners in the registry |
| Wallet | Idea account keyed by work ID (OIP-8) |
| License | `license`, per version |
| Relationship (parents and siblings) | `parents` and `cites` in the version; approved weights and other links in the registry (OIP-14) |
| Content | `content_refs`, by hash |
| Reputation | Not part of an idea; reputation belongs to people (OIP-9). Ideas receive reviews |
| Reviews | Separate ideas of type `review` that target this one |
| `types` | `type` plus the community the work belongs to |
| `source` (previous version) | `previous` |
| `supporting_files` | `content_refs` |
| `authors` | `attribution`, kept separate from `owners` and payout rights (OIP-10) |
| `events` | Ledger transactions on the work |
| `keywords` | `domain_tags` |

### Why the work ID is the first version's ID

It needs no counter or extra randomness, it is known as soon as the first version is built, and it is unique because the first version includes its registrant and owners.

### Why JSON with RFC 8785

The ID must be identical across implementations and languages. RFC 8785 is a published standard for canonical JSON with libraries in common languages, and JSON is readable in the public GitHub ledger. It resolves the canonical-serialization question for idea versions; whether to use it for every transaction is left to OIP-13.

### Why owners and registrant are inside the version

Owners sign the version, so the signed record must say who claims what. Including the registrant also means that two people registering identical content get distinct IDs, so that neither can block the other by registering first; duplicates are then decided at admission, where a person can judge them.

### Why license changes need a new version

OIP-10 says a license never changes retroactively. Attaching the license to the version makes that automatic: what was released under CC-BY stays CC-BY.

### Open questions

1. Should preprints and their journal versions be one work with two versions, and who decides?
2. Should `hypothesis` ideas be allowed to mint, given that they are cheap to write?
3. Are the size limits (300 characters, 5,000 characters, 64 KiB) right?
4. Should datasets and code be required to carry a content hash even when imported?

## Backwards Compatibility

Replaces the idea definition of the GIP repository (`README.md` and `models/Idea.py`, 2017–2018) and §4.1 of the OSO Idea Platform whitepaper v0.3. The GIP description of an idea as an entity that generates new information is kept. The idea fields in the v1 design doc's Core concepts table are superseded by section 4. No deployed system depends on the earlier definitions.

## Test Cases

### Version ID

This first version of a preprint (its content hash is the SHA-256 of the four bytes `test`, for illustration):

```json
{"abstract":"A short example.","ai_disclosure":{"description":"","level":"none"},"attribution":[{"name":"A. Researcher","orcid":"0000-0002-1825-0097"}],"cites":[],"content_refs":[{"hash":"sha256:9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08","media_type":"application/pdf","role":"main","uri":"https://arxiv.org/abs/0000.00000"}],"domain_tags":["ml.peft"],"external_ids":{},"license":"CC-BY-4.0","owners":[{"identity":"0x0000000000000000000000000000000000000001","share_bps":10000}],"parents":[],"previous":null,"registrant":"0x0000000000000000000000000000000000000001","schema":"oso.idea/1","targets":{},"title":"Example idea","type":"preprint"}
```

is already in RFC 8785 form (665 bytes) and has the version ID:

```
sha256:88bfa9d861437065384528061580caf48eb06749ab9c116004076b854bacfa00
```

Because it is a first version, its work ID is the same value. Every field is present, with empty values where the type allows (`targets` is `{}` because a preprint targets nothing).

### Required cases

- [ ] Reordering the keys of a version, or adding whitespace, does not change its ID.
- [ ] A version with an unknown or missing field is rejected.
- [ ] A registration whose stated ID differs from the recomputed ID is rejected.
- [ ] Changing owners through a registry update leaves every version ID and the work ID unchanged.
- [ ] Changing the license requires a new version, and the earlier version still shows the earlier license.
- [ ] A second version naming the same `previous` as an existing version is rejected.
- [ ] Importing a DOI that already belongs to a work is rejected.
- [ ] A new submission with no hashed `main` content reference is rejected.
- [ ] A `review` without exactly one `reviews` target is rejected.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Content swapped behind the same link after signing | Content hash in the signed version; mismatch is detectable by anyone |
| Registering someone else's work as one's own | Signatures show who registered it; plagiarism checks at admission and challenges (OIP-11) |
| Blocking a real author by registering their content first | Version IDs include registrant and owners, so the real author can still register; admission decides duplicates |
| Duplicate imports splitting credit | External-identifier index rejects a second work for the same DOI, arXiv ID or OpenAlex ID |
| Oversized or spam metadata | Field and size limits; submission stake and pre-screen (OIP-8, OIP-11) |
| Two implementations computing different IDs | One canonical serialization (RFC 8785), one hash function, exact field list |
| Weakening of SHA-256 in the future | The `sha256:` prefix lets a later schema version name another algorithm |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
