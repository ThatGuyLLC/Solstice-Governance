# Solstice Governance Repository

> **This repository replaces the [Filecoin Plus Allocator Governance repository](https://github.com/filecoin-project/Allocator-Governance).** The Filecoin Plus (FIL+) DataCap program is being sunset. Solstice is its successor: instead of allocating DataCap to onboard capacity, the network funds measured, paid storage *service* through a block-reward split. This repository is the operational layer of that program.

## Start here

- **The protocol rules that govern this program live in the FIP:** [**FIP-0118 — (Solstice) Deprecate FIL+ and Fund Services via Block-Reward Split**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md). The FIP fixes *what is enforced*; this repository holds *how it is operated*. Throughout these pages, any rule fixed by the FIP is quoted verbatim in a blockquote and attributed to FIP-0118.
- **For the why:** [Solstice — Towards a Filecoin Service Economy](https://www.filecoin.io/blog/Solstice-Towards-a-Filecoin-Service-Economy), the blog post breaking down the shift from a storage-capacity onboarding phase to a service-oriented economy.
- **Discussion:** [FIP-0118 discussion thread](https://github.com/filecoin-project/FIPs/discussions/1249).

> From the FIP-0118 Simple Summary:
>
> > This proposal removes the Filecoin Plus datacap system and replaces it with an automated mechanism. All new storage sectors receive equal 10× power at the protocol level. The block reward is then split into three streams—consensus (paid to miners), service (paid to registered orchestrators), and burn—with weights that adjust based on verifiable on-chain payment volume from Filecoin Pay.

## How this document maps to the repository

Each section below is a separate page, ordered by its number, so a reader interested in only one topic can open that page alone. File names carry the section number so they sort in reading order.

| Section | Repository file | Covers |
| :-- | :-- | :-- |
| 1. README | [`README.md`](README.md) | Purpose, references, section map |
| 2. Solstice Program Governance | [`docs/02-solstice-program-governance.md`](docs/02-solstice-program-governance.md) | Governance tiers & Safes · SWA tier · SRA tier (+ Orchestrator Registry) · Parameters · Safety and rotation playbook |
| 3. Orchestrator Operational Guidelines | [`docs/03-orchestrator-operational-guidelines.md`](docs/03-orchestrator-operational-guidelines.md) | Orchestrator guidelines · Declaration, verification and escalation |
| 4. Program Change Log | [`docs/04-changelog.md`](docs/04-changelog.md) | Rule-change history with community issues & FIP links |

> Issue templates, application forms, and workflow automation are out of scope for this draft.

## 1.1 Purpose

This repository is the operational layer of the governance defined in the Solstice FIP. The FIP fixes what is enforced: the two governance tiers and their powers, the change lifecycle, and the invariants the threat model relies on. This repository holds how those tiers operate: who stands behind each Safe, the procedures for admission, disputes, verification, and rotation, and the parameters designed to iterate without a FIP.

## 1.2 What lives here

- The wallet addresses of the organizations behind each tier's Safes.
- Tasks and actions for each ecosystem actor, inclusive of the operational guidelines for each role, and admission and removal criteria for Orchestrators.
- The dispute procedure for contested bindings, and the verification playbook.
- Measurement-rule declarations for each orchestrator.
- The registry of admitted Orchestrators and the change log of program-rule updates.
- The versioned reference indexer that recomputes FPV from public settlement events.
- The durations of `SRA_CANCEL_HOLD`, `POST_PERIOD`, and `VERIFICATION_WINDOW`.

## 1.3 What requires a FIP

Rules — meaning contract code and every invariant the threat model relies on — change only through a FIP: the existence and powers of the two tiers; the two-Safes rule itself; the approve-and-hold lifecycle; the per-change FIP requirement on discretionary SWA writes; the distinction between mechanism-executed updates and discretionary writes; the (payer, operator) uniqueness invariant; the L1 conditions (sum of weights at most 1, stream and recipient caps, the `SWA_TIMELOCK` value); upgrades to either contract's code; and the backstop for replacing a tier that can no longer act. These sit above the tiers they constrain deliberately: a rule that binds a governance tier cannot live in a repository that this specific tier edits.

> FIP-0118 fixes the two-Safes rule verbatim:
>
> > "Both Safes approve, the change is published and held, and either Safe can cancel during the hold."

Gate parameters (step size, volume targets, escalation ratio) are also not repository content. They live in the SWA, and changing them is a `SetGateParams` write requiring both SWA Safes and a published FIP.

## 1.4 Changing this repository

Changes happen by pull request with public review — covering, but not limited to, merge rights, review windows, and whether any section requires sign-off from both tiers. Every change to the program rules is recorded in the [Program Change Log](docs/04-changelog.md), which links the originating community issue and, where applicable, the resulting FIP.

## 1.5 Reference Links (TBD)

- FIP text: [FIP-0118](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) *(number pending final assignment)*
- Governance meetings: TBD (not mandatory)
- Reference indexer: TBD
- Settlement data and dashboards: TBD

---

*Solstice Governance Repository — operational layer of [FIP-0118](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md).*
