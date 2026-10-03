---
oip: 15
title: AI services, models and costs
description: Defines how OSO chooses and runs AI models, including locally hosted ones, how outputs are recorded, and who pays for AI compute.
author: Gajendra Jung Katuwal (@himalayajung)
discussions-to: TBD (pull request URL once opened)
status: Draft
type: Standards Track
category: Module
created: 2026-10-02
requires: 12, 13, 14
---

## Abstract

OSO uses AI for pre-screening submissions, assessing how ideas depend on each other, tagging domains, drafting review notes and answering questions about ideas. This OIP keeps OSO independent of any single model or provider. AI is a replaceable module; each community pins the models it uses per task; and every output is recorded with its model, version and prompt so that replay never depends on a model. Tasks that can affect payouts must be checked by two models from different families, and disagreements go to people. Models may run through hosted APIs or be hosted locally with open-weight models. The OSO fund pays for AI processing of submissions, so authors never pay a processing fee; in v1, before OSO has outside value, the founding team carries the real-money cost and publishes the budget.

## Motivation

[OIP-11](./oip-11.md) and [OIP-14](./oip-14.md) depend on AI outputs, [OIP-12](./oip-12.md) defines an AI-services module slot, and [OIP-13](./oip-13.md) requires AI outputs to be recorded. None of them says which models are used, how they are chosen and run, how unpublished work is protected when sent to a model, or who pays. AI compute is a real-money cost that exists from the first day, long before OSO tokens have outside value.

## Specification

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

### 1. Model independence

1. AI services MUST be implemented as modules (OIP-12). The protocol MUST NOT depend on a particular model, provider or hosting method.
2. Each community setup MUST pin, for each AI task, the model identifier and version, the provider or host type, the prompt template (by identifier and hash), and the generation settings.
3. A change of pinned model MUST take effect only from the setup's activation block, and MUST NOT change any recorded output or past decision.
4. An AI service adapter SHOULD speak a widely supported chat-completion interface, so that hosted and locally hosted models can be swapped without code changes.

### 2. Task classes

| Class | Tasks | Requirement |
| --- | --- | --- |
| A: affects payouts or admission | Intrinsic assessments that can lead to payout edges (OIP-14 section 4), including those for new submissions and the curated slice; pre-screen reports, including their suggested parents and weights (OIP-11); plagiarism judgments | Two models from different model families MUST assess independently. If their scores differ by more than the community's disagreement threshold, the item MUST be marked for human attention. At least one of the two SHOULD be an open-weight model. |
| B: informs people, moves no value | Domain tagging, assessments for the read-only import tier (imported works outside the curated slice, which have no payout edges), review-note drafts, suggested parents shown while an author drafts a submission | One model is sufficient. |
| C: retrieval | Similarity search, duplicate detection | Embedding models; no generative model required. |
| D: conversation | Chat about ideas | Any pinned model; outputs are not recorded on the ledger (they are not inputs to any decision). |

A cheap Class C check (for example, embedding similarity against existing ideas) SHOULD run before any Class A model on a new submission.

### 3. Recorded outputs

For every Class A and Class B output, the ledger MUST record (as an AI input transaction, OIP-13 section 6):

- the task and the items assessed;
- the model identifier and version; for open-weight models, the hash of the weights file;
- the provider or host type (hosted API, local, community node);
- the prompt template identifier and hash, and the generation settings;
- references to the input content;
- the output;
- token counts, and the cost where known.

Replay MUST read these records and MUST NOT re-run any model. GPU inference can vary slightly between runs even with fixed settings, so the recorded output, not a re-run, is authoritative.

### 4. Deployment options

A community MAY use any combination of:

1. **Hosted APIs** from model providers. Before sending unpublished work to a hosted API, the operator MUST check the provider's data-use terms, and MUST NOT use a provider that trains on submitted content unless the authors have agreed.
2. **Locally hosted open-weight models**, run by the operator or the community on its own hardware. Commonly used tools include:

   | Tool | Typical use |
   | --- | --- |
   | Ollama | Simple local serving of open-weight models on a laptop or server |
   | llama.cpp | Lightweight inference of quantized models, on CPU or GPU |
   | vLLM | High-throughput serving on GPU servers, for bulk tasks such as the read-only import tier |
   | LM Studio | Desktop application for running and testing models locally |

   These tools can expose a chat-completion interface compatible with the adapter in section 1. Open-weight model families such as Llama, Qwen, Mistral, Gemma and DeepSeek can be served this way; communities choose specific models and versions in their setup. For a local model, the hash of the weights file MUST be recorded (section 3), so that anyone can obtain the same weights and audit an output.
3. **Community AI-service nodes** (later): independent operators run models and are paid per task (section 6). This option requires a later OIP for audits and payment.

**Privacy option.** Authors MAY require that their unpublished submission be processed only by locally hosted models. The operator MUST honour this choice; if no local model is pinned for a required task, the submission MUST wait (OIP-11 Waiting state) rather than be sent to a hosted API.

### 5. Who pays

1. **v1.** In v1, OSO tokens have no outside value, so the real-money cost of AI compute MUST be covered by the founding team, grants or donated compute credits. The operator MUST publish the AI budget and actual spending.
2. **Submissions.** AI processing of a submission (Class A and B tasks) MUST be paid from the OSO fund. Authors MUST NOT be charged an AI processing fee. The fund's 30% share of each mint ([OIP-8](./oip-8.md) section 4), which Proof of Idea allocated to storage, development and maintenance, is the intended source.
3. **Abuse.** The submission stake (OIP-8 section 8) of a submission rejected as spam goes to the OSO fund, offsetting the AI cost it caused.
4. **Chat.** Chat MUST have a free tier paid from the OSO fund, limited per identity per round. Users MAY go beyond the free tier by paying in OSO or by supplying their own API key or local model.
5. **Community upgrades.** A community that pins models costing more than the network default MUST pay the difference from its own treasury.
6. **AI-service market (later).** Community AI-service nodes MAY be paid per task in OSO, once a later OIP defines how their outputs are audited.

### 6. Budget and abuse controls

1. The operator MUST set an AI budget per round and a per-identity submission limit.
2. When the round's budget for Class A tasks is exhausted, new submissions MUST move from Submitted to Waiting (OIP-11 section 1) until the next round. Required checks MUST NOT be skipped to save cost.
3. Class C filters MUST run before Class A models.
4. AI spending per round, by task class, model and provider, MUST be published.

### 7. Handling submitted content

1. Content sent to a model MUST be treated as data. Prompts MUST separate instructions from submitted content and MUST request structured output.
2. Outputs that do not match the required structure MUST be rejected and the task retried or marked for human attention.
3. Text in a submission that appears to be directed at a model (for example, instructions to rate the work highly) MUST be reported to validators.

### 8. Parameters

| Parameter | Meaning | v1 value | Set by |
| --- | --- | --- | --- |
| Pinned models | Model and version per task class | Chosen at setup | Community |
| Disagreement threshold | Score difference that sends a Class A item to people | TBD bps | Community |
| Free chat allowance | Chat requests per identity per round | TBD | Community |
| AI budget | Maximum AI spending per round | Published by operator | Operator (v1) |
| Submission limit | Submissions per identity per round | TBD | Community |

## Rationale

- **Model independence.** Models improve and change quickly; hard-wiring one would repeat the 2018–2020 pattern of rebuilding around each new component. Pinned versions and recorded outputs let models change without breaking replay.
- **Two models for Class A.** A single model's blind spots and biases would otherwise flow straight into payouts. Using different model families, at least one open-weight, combines quality with auditability.
- **Local hosting.** It lets communities protect unpublished work, control costs at high volume, and audit outputs with exactly the same weights.
- **The fund pays, not authors.** Proof of Idea's promise is that publishing earns rather than costs. A processing fee would reintroduce a pay-to-publish barrier.
- **Waiting instead of skipping checks.** A budget shortfall must never weaken spam and plagiarism protection.
- **Prompt injection.** Submissions are untrusted input to models that inform admission and payouts; treating them strictly as data and checking structure are basic defenses.

### Further research

AI assessments of how much one idea depends on another are judgments, and their quality is not yet established. Research needed to improve idea attribution is listed in OIP-14, under Further research.

### Open questions

1. Which models should the first community pin for each task class?
2. Value of the disagreement threshold.
3. How should community AI-service nodes be audited and paid?
4. Should chat transcripts ever be recorded, for example when a chat leads to a new submission?

## Backwards Compatibility

New. Complements the AI rules in the OSO v1 design doc ("Role of AI").

## Test Cases

To be added. Required cases:

- Replay with all model access disabled reproduces the same state hash.
- A Class A item whose two model scores differ by more than the threshold is marked for human attention.
- A submission marked local-only is never sent to a hosted API; with no local model pinned, it waits.
- When the Class A budget is exhausted, new submissions wait and no required check is skipped.
- An output that does not match the required structure is rejected.

## Reference Implementation

None yet.

## Security Considerations

| Risk | Mitigation |
| --- | --- |
| Prompt injection in submissions | Content treated as data, structured outputs, reporting to validators, two-model checks, human approval for payouts (OIP-14) |
| Bias or error in a single model | Two model families for Class A; disagreements go to people |
| Vendor lock-in or a provider withdrawing a model | Model-independent adapter; open-weight option; recorded outputs keep past decisions valid |
| Leaking unpublished work | Provider data-use checks; local-only processing option |
| Draining the AI budget with spam | Stakes, per-identity limits, cheap filters first, per-round budget with waiting |
| Unverifiable outputs from AI-service nodes | Not allowed until a later OIP defines audits |

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
