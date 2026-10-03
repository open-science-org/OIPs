# OSO Idea Proposals (OIPs)

OSO Idea Proposals (OIPs) describe standards, modules, interfaces and processes for the Open Science Organization (OSO). An OIP can be a new concept, a protocol rule, a module design, or anything that helps create an open, democratic and efficient scientific ecosystem.

The process is modeled on Ethereum's EIPs. Read [OIP-0: OIP purpose and guidelines](OIPS/oip-0.md) before writing one.

## How to propose an OIP

1. Discuss the idea first in an issue in this repository.
2. Copy [oip-template.md](oip-template.md), fill it in, and open a pull request with the file named `oip-draft_short_title.md`.
3. An editor checks the format, assigns a number, and merges it as Draft. Discussion continues on the pull request.

## OIPs

### Meta

| Number | Title | Status |
| --- | --- | --- |
| [0](OIPS/oip-0.md) | OIP purpose and guidelines | Living |
| [2](OIPS/oip-2.md) | Funding application | Stagnant |
| [3](OIPS/oip-3.md) | Funding OSO | Stagnant |

### Standards Track

| Number | Title | Category | Status |
| --- | --- | --- | --- |
| [8](OIPS/oip-8.md) | Token structure | Core | Draft |
| [9](OIPS/oip-9.md) | Reputation and expertise | Module | Draft |
| [10](OIPS/oip-10.md) | Ownership and identity | Core | Draft |
| [11](OIPS/oip-11.md) | Submission routing | Module | Draft |
| [12](OIPS/oip-12.md) | Module interface and community setups | Core | Draft |
| [13](OIPS/oip-13.md) | Public ledger and migration path | Core | Draft |
| [14](OIPS/oip-14.md) | Idea attribution and value flow | Core | Draft |
| [15](OIPS/oip-15.md) | AI services, models and costs | Module | Draft |

### Informational

| Number | Title | Status |
| --- | --- | --- |
| [1](OIPS/oip-1.md) | IPFS integration and platform UI | Stagnant |
| [4](OIPS/oip-4.md) | Basic validation layer and validator merit | Stagnant |
| [5](OIPS/oip-5.md) | Custom validation layers | Stagnant |
| [6](OIPS/oip-6.md) | IdeaBoard | Stagnant |
| [7](OIPS/oip-7.md) | Publishing as a cascade of TCRs | Stagnant |

OIPs 1–7 were originally written as GitHub issues (2017–2019). Their files hold a summary and a link to the original issue and its discussion.

## Repository layout

| Path | Contents |
| --- | --- |
| `OIPS/oip-N.md` | One file per OIP |
| `assets/oip-N/` | Images, diagrams and auxiliary files for OIP N |
| `oip-template.md` | Template for new OIPs |
| `LICENSE.md` | CC0 1.0, which applies to all OIP text in this repository |
