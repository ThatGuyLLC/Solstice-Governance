# 3. Orchestrator Operational Guidelines

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/irenegia/FIPs/blob/irenegia-solstice/FIPS/fip-0118.md) and quoted verbatim where referenced.

This section consolidates what is expected of Orchestrators (the operational guidelines) and how declarations are verified and escalated.

**Contents**

- [3.1 Policies](#31-policies)
- [3.2 Orchestrators Tasks and Actions](#32-orchestrators-tasks-and-actions)
- [3.3 Monitoring tools & references](#33-monitoring-tools--references)
- [3.4 Declaration, verification and escalation](#34-declaration-verification-and-escalation)

The service stream pays for measured, paid storage service. These rules give orchestrators predictability about what is expected and what the consequences of a breach are, and give the network confidence that the number driving the gate and the shares reflects reality. The standing test for any action in the program: **does this volume represent a client paying for storage service on Filecoin?** If the answer is yes, the orchestrator is operating within both the rules and their intent.

Orchestrators are free to run their businesses autonomously. The program does not review their pricing, their client selection, or their operations. To maintain trust in the measurement layer, all orchestrators are expected to adhere to the criteria below. These may be revised over time through the standard PR process (and logged in the [Program Change Log](04-changelog.md)).

- **Upfront disclosures.** Before admission, or a binding change, an orchestrator discloses all addresses it controls, has a financial stake in, or is strongly connected to by other means, including any common-control relationship with a payer, an operator, or another orchestrator.
- **Measurable settlement.** Orchestrators route paid storage service revenue through admitted Filecoin Pay contracts in admitted stablecoins or FIL, and help their clients settle the same way. Volume that bypasses admitted rails is invisible to the mechanism and counts for nothing.
- **Responsiveness.** Orchestrators respond to dispute and verification requests within the response window (72 hours) and keep their contact information in this repository current.
- **No self-dealing.** An orchestrator does not settle volume between parties under common control, and does not declare a (payer, operator) pair without a client relationship behind it. The disclosure duty above exists so that common control is visible before it becomes a finding post-verification.
- **Good-faith declarations.** Declarations are accepted by default. The program extends trust upfront because every posted figure is recomputable from public events; misreporting is mechanically detectable and is grounds for freeze or removal.
- **Self-monitoring.** Orchestrators are expected to monitor their own recomputed FPV against posted figures during each verification window, and to flag discrepancies rather than wait for them to surface as findings. A deliberate distinction from the Fil+ program: there are no mandatory governance calls. The duty is to the public record, not to a meeting cadence.

## 3.1 Policies

Each policy notes where it is enforced:

1. **FIP** — protocol invariant, changes require a FIP;
2. **Contract** — enforced by SRA or f02 code;
3. **Repository** — program rule, changes by PR.

| # | Policy | Enforced |
| :-- | :-- | :-- |
| 1 | The program is open to any entity that routes paid storage service revenue through admitted settlement rails on behalf of clients. Admission is discretionary in Phase 1, scored against the admission rubric; a future FIP makes admission permissionless in Phase 2. | Repository |
| 2 | Qualifying volume is settlement through admitted Filecoin Pay contracts, in admitted stablecoins or FIL converted off-chain via the reference indexer using public fee-auction prints (`MIN_LOT`, `PRICE_BAND`), attributable to the orchestrator's registered (payer, operator) pairs. Nothing else counts. | Contract |
| 3 | A (payer, operator) pair binds to exactly one orchestrator. Registering a pair already bound elsewhere reverts. Clients are free to work with multiple orchestrators across different operator relationships; the pair, not the client, is the unit of attribution. | FIP |
| 4 | Volume counts only for pairs registered before the settlement occurs. Retroactive attribution is not accepted. | Repository |
| 5 | Each quarter, orchestrators post FPV within `POST_PERIOD`. A value neither posted nor corrected during `VERIFICATION_WINDOW` binds at zero. Bound values are final; a successful appeal affects later quarters only. | FIP and contract |
| 6 | Within 7 days of admission, each orchestrator publishes its declaration file in this repository: a short description of its service, contact information, and its registered pairs, including any pointer to service-contract metadata. | Repository |
| 7 | An orchestrator with no settled volume for **[TBD timeline]** consecutive quarters enters registry review and may be removed. Removal on inactivity is registry hygiene: an idle registration adds attack surface without adding measurement value. | Repository |
| 8 | No registry address participates in either governance tier, as a Safe or as a key holder within one. | FIP |

Policies 3, 5, and 8 restate FIP-0118 invariants verbatim:

> "Each (payer, operator) pair is bound to at most one orchestrator, so a registration that duplicates an existing binding reverts."
>
> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter." … "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."
>
> "No address in the registry may participate in either contract governance tier, as a Safe or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

## 3.2 Orchestrators Tasks and Actions

> These are operational tasks and actions; each is a collapsible dropdown, and they may be updated by raising an issue in the present repository.

<details>
<summary><strong>3.2.1 — Deal making</strong></summary>

> **Cadence:** continuous · **On-chain call:** none

Orchestrators set their own pricing, choose their own clients, and structure their deals, subsidies, and operations as they see fit. The program does not review or approve commercial terms, client selection, or operations. The only constraint is measurability: paid storage-service revenue must settle through admitted Filecoin Pay rails on registered (payer, operator) pairs so it can be counted.

</details>

<details>
<summary><strong>3.2.2 — Orchestrator controlling wallet — identity and matching</strong></summary>

> **Cadence:** at admission + on a need basis (replace) · **On-chain call:** `Replace(old, new)` (rotation)

Each orchestrator is represented on-chain by a single controlling wallet: the address registered in the SRA registry.

1. Register one clearly-owned address as your controlling wallet — the single identity that receives the service-stream share from f02, and calls `RegisterPairs`/`PostVolume`.
2. As part of the admission process (see [§2.3 SRA Governance Tier](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)), record in your GitHub declaration issue which address runs your orchestrator operation. Your entry then appears in the [Orchestrator Registry](02-solstice-program-governance.md#2316-orchestrator-registry-admitted-orchestrators).
3. Hold it as a multisig (e.g., a Safe) with hardware-key signers, rather than a single externally-owned key (recommended).
4. Rotate a compromised or changed address via `Replace(old, new)` (a registry change); f02 pays the new address from the effective epoch, so no payment is missed.

</details>

<details>
<summary><strong>3.2.3 — Linking deals to Filecoin Pay services</strong></summary>

> **Cadence:** per deal, before settlement · **On-chain call:** `RegisterPairs`

Volume counts only when it settles on an admitted Filecoin Pay contract, in an admitted stablecoin or FIL, on a rail whose (payer, operator) pair is registered to you *before* the settlement occurs. To link a deal:

1. Set up the client's payment on an admitted Filecoin Pay contract (admitted-contract list).
2. Create the rail with the operator you will register.
3. Register the (payer, operator) pair via `RegisterPairs` before the first settlement. Retroactive attribution is not accepted; a pair already bound elsewhere reverts (uniqueness invariant).
4. The client settles on-rail (`RailSettled` or one-time payments); those settlements accrue to your FPV.

> **Note:** volume on unregistered pairs, off admitted rails, or in non-admitted assets will not be accounted for in the FPV measurement.

</details>

<details>
<summary><strong>3.2.4 — Measurement and receiving funds</strong></summary>

> **Cadence:** measurement quarterly · payout each epoch · **On-chain call:** `PostVolume` (measurement)

1. Understand your figure: FPV_i(Q) is the value settled on your registered (payer, operator) rails during a given quarter; amounts are denominated in USD — admitted stablecoins at face value, FIL converted off-chain via the reference indexer using public fee-auction prints (`MIN_LOT`, `PRICE_BAND`).
2. Declare how your volume is measured (your pairs, optional service-contract metadata, booked-revenue basis).
3. Funds are received automatically: f02 pays your controlling wallet its share of the service stream (w1) directly, every epoch — no invoice and no claims are required.

The following rule establishes how the share is set: `SplitRule` = your bound FPV_i(Q) ÷ AggregatedFPV(Q), written into f02 once per quarter via `SetShares`.

A quarter with no bound volume yields a zero share for that quarter; your wallet is never debited.

</details>

<details>
<summary><strong>3.2.5 — Treasury function and wallet-safety best practices</strong></summary>

> **Cadence:** continuous · **On-chain call:** none (off-protocol)

Rewards land in your controlling wallet each epoch; deploying them against your mandate (client acquisition, subsidies, integrations, migrations, infrastructure, etc.) is your responsibility and lies outside the protocol. Recommended practices:

- **Custody:** hold reserves in a multisig (e.g., Safe) with hardware-key signers; never keep the treasury on a single hot key.
- **Segregation:** keep a cold reserve (offline/hardware, high signing threshold) separate from a small hot operating wallet; sweep excess from hot to cold.
- **Key hygiene:** never expose or share private keys or seed phrases, and never place them in code, configs, or shared documents; if exposure is even suspected, rotate immediately (`Replace`).
- **Least privilege:** where you use exchanges or custodial APIs, prefer read-only keys and disable withdrawal/trading scopes you don't need.
- **Monitoring & recovery:** alert on inflows/outflows, name an accountable key-custody owner, and keep a documented, tested recovery path.
- **Conversion discipline:** convert only what runway requires; hold the reserve.

</details>

<details>
<summary><strong>3.2.6 — Reporting and reconciliation</strong></summary>

> **Cadence:** quarterly (`POST_PERIOD`, then `VERIFICATION_WINDOW`) · **On-chain call:** `PostVolume`

1. Within `POST_PERIOD`, post your figure via `PostVolume` as a single USD-denominated total, with FIL volume converted off-chain by the indexer per the fee-auction pricing rule.
2. During `VERIFICATION_WINDOW`, recompute your own FPV from public settlement events using the reference indexer and reconcile it against what you posted; flag any discrepancy proactively.
3. Keep your declaration file (pairs, contact, measurement rules) current.
4. Remember the failure mode: a value neither posted nor corrected binds at zero (a conservative under-count); bound values are final, and a successful appeal affects later quarters only.

FIP-0118 fixes the posting period and the no-clawback rule:

> "Posting: each Orchestrator posts FPV_i(Q) to the SRA during the posting period (E, E + POST_PERIOD]."
>
> "There is no retroactive clawback: a misreport discovered after binding is grounds for audit, freeze, or removal."

</details>

<details>
<summary><strong>3.2.7 — Declarations, verification, and appeal</strong></summary>

> **Cadence:** as triggered · **On-chain call:** none (governance-side `CorrectVolume` / `ReassignBinding`)

Declarations are accepted by default; verification is exception-based and dispute-driven, never pre-clearance. If a binding is contested or a figure disputed, the process runs through the dispute resolution process in [§3.4.2](#342-dispute-resolution-for-contested-bindings), and any on-chain outcome is held under `SRA_CANCEL_HOLD`. Appeals of a `CorrectVolume` follow the same evidence process and can affect only later quarters, since bound values are final.

</details>

## 3.3 Monitoring tools & references

*(links to be updated)*

These tools are meant to ease the monitoring of the program's activity. Orchestrators use them to check their recomputed FPV against what they posted; SRA Governance and any community observer runs the same tools for oversight. Every verification action and its conclusion must be reproducible from these public sources.

- Versioned reference indexer: *(links to be added)*
- Governance & signing interface: *(links to be added)*
- Settlement data and dashboards: *(links to be added)*
- Registry state, admitted-stablecoin whitelist, and admitted orchestrators: see the [Orchestrator Registry](02-solstice-program-governance.md#2316-orchestrator-registry-admitted-orchestrators) *(on-chain links to be added)*
- Filecoin Pay contracts: *(links to be added)*

---

## 3.4 Declaration, verification and escalation

### 3.4.1 Binding declarations and verification

Declarations are accepted by default, and rely upon a mix of on-chain verifiable metrics and other KPIs/SLAs. Verification is dispute-driven: it is triggered only based upon provable gaps highlighted by community members. A false claim surfaces when the legitimate orchestrator's registration and performance cannot be proven via either on-chain transactions or credible probing documentation for off-chain transactions. Verification methods per our integrity guidelines are set out below.

FIP-0118 fixes the consequence of a misreport discovered after binding:

> "There is no retroactive clawback: a misreport discovered after binding is grounds for audit, freeze, or removal."

### 3.4.2 Dispute resolution for contested bindings

1. The contesting party opens an issue in this repository identifying the pair and its claim.
2. SRA Governance requests evidence of the client relationship from both parties. Acceptable evidence, in increasing strength: business records showing the engagement; client confirmation through a verifiable channel; a signed client attestation (a message signed by the payer wallet naming the orchestrator).
3. Resolution target timeline: 7 days. The outcome (reassignment, removal, or no action) executes on-chain, subject to the hold, and is recorded in the issue.

A contested `CorrectVolume` follows the same evidence process. Per the FIP, bound values are final, so the appeal outcome can only affect later quarters:

> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter."

### 3.4.3 Verification, freeze, and removal

One inversion from the allocator program, and it is deliberate: allocator pathways were audited on a schedule, at each DataCap refresh. Orchestrator verifications are exception-based, per the FIP, and are never pre-clearance. A scheduled audit cycle would rebuild the review pipeline Solstice deprecates. A verification begins only when a trigger fires: a public-data anomaly, a community report filed as an issue, or a contested binding escalated from the dispute process ([3.4.2](#342-dispute-resolution-for-contested-bindings)).

**Grounds for freeze or removal:** wash trading, self-dealing (settling volume between parties under common control), misreported FPV (mechanically detectable, since FPV is recomputable from public events), and binding fraud.

### 3.4.4 What triggers a verification

- A public-data anomaly surfaced by the quarterly FPV recomputation or by monitoring (volume spikes at quarter/gate boundaries, figures that clear the target by a hair, volume with no matching storage activity).
- A community report filed as an issue in this repository.
- A contested binding escalated from the dispute process.

### 3.4.5 Escalation

1. **Trigger** → run the relevant on-chain checks (see [§3.2](#32-orchestrators-tasks-and-actions)); the recomputation is deterministic and public.
2. **Request evidence** if an anomaly holds: open a public issue and request on- and off-chain evidence; the orchestrator answers within the response window (72 hours).
3. **Interim freeze** where funds or the gate are at live risk: `Freeze(orch)` — the Orchestrator is suspended (cannot `RegisterPairs`/`PostVolume`; share = 0; excluded from `AggregatedFPV` for any quarter closing while frozen), its bindings stay put, and other Orchestrators keep being paid.
4. **Decide:** reinstate (unfreeze), reassign a binding, or `Remove(orch)`.
5. All on-chain actions run through the standard registry-change flow under **`SRA_CANCEL_HOLD`** (see [§2.3](02-solstice-program-governance.md#23-sra-governance-tier--tasks-and-actions)) and are recorded, with rationale, in the issue.

### 3.4.6 Cross-cutting verification measures

- **Reproducibility:** every verification conclusion must be reproducible from public data via the versioned reference indexer; findings cite the events and blocks used.
- **Sanctions cadence:** re-screen registered addresses on a fixed cadence and at each admission, not only on trigger.
- **Conflict attestations:** collect a conflict-of-interest / no-common-control attestation at admission and on any roster or binding change.
- **Verification trail:** triggers, evidence, decisions, and on-chain actions are all logged in this repository.

**Procedure (recap):**

1. Public issue in this repository.
2. Orchestrator response window (72 hours).
3. Governance decision.
4. On-chain action subject to `SRA_CANCEL_HOLD`.

---

← Previous: [2. Solstice Program Governance](02-solstice-program-governance.md) · [Back to README](../README.md) · Next: [4. Program Change Log](04-changelog.md) →
