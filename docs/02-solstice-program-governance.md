# 2. Solstice Program Governance

> Part of the [Solstice Governance Repository](../README.md). Protocol rules are fixed by [**FIP-0118**](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) and referenced throughout. This section describes *how* the program is governed and operated.

This section consolidates the operating manual for the two governance tiers: the standards behind each Safe, the tasks and actions each tier performs, the registry of admitted Orchestrators, the program parameters, and the safety and rotation playbook.

**Contents**

- [2.1 Governance Tiers and Safes](#21-governance-tiers-and-safes)
- [2.2 SWA Governance Tier — Tasks and Actions](#22-swa-governance-tier--tasks-and-actions)
- [2.3 SRA Governance Tier — Tasks and Actions](#23-sra-governance-tier--tasks-and-actions) (including the Orchestrator Registry at 2.3.16)
- [2.4 Parameters](#24-parameters)
- [2.5 Safety and rotation playbook (both tiers)](#25-safety-and-rotation-playbook-both-tiers)

---

## 2.1 Governance Tiers and Safes

Solstice governance is split across two contracts, each operated by a tier composed of two independent organization multisigs (Safes): **Tier 1, the SWA Governance Tier (§2.2)**, and **Tier 2, the SRA Governance Tier (§2.3)**. FIP-0118 fixes the separation and the approval structure:

> "No address in the registry may participate in either contract governance tier, as a Safe or as a key holder within one: a party that set weights could raise its own share, and one that ran the registry could admit itself or block competitors."

### 2.1.1 Standards for a governance Safe (both tiers)

- The organization is publicly identified, has a track record in Filecoin governance or engineering, and is independent of the other organization in the same tier.
- Recommended, not protocol-enforced: an internal threshold of at least 2, and hardware key custody.
- No orchestrator role for the organization or any of its key holders. Per the FIP, no registry address may participate in either tier, as a Safe or as a key holder within one.
- Internal member rotation is org-internal and immediate; it requires no governance action.

### 2.1.2 Approval and cancellation rules (both tiers)

These rules are fixed by the FIP and restated here for operators.

> FIP-0118, on the change lifecycle:
>
> > "Every discretionary change follows the same path: both Safes approve, the change is published and held, and either Safe can cancel during the hold."

**Mechanism-executed updates** take no approval and are not cancellable. The quarterly gate check and the `SetShares` recompute run permissionlessly through entry points that can emit only the value the mechanism computes; their integrity guard sits upstream in the FPV verification window. The tiers exercise no judgment over these updates because there is no judgment to exercise.

> FIP-0118 on why mechanism writes cannot be cancelled:
>
> > "Mechanism-executed updates cannot be cancelled: the gate's quarterly w2 write queues in f02 under SWA_TIMELOCK for visibility, but it is tagged as a mechanism write at queue time and CancelPending rejects it."

**Discretionary changes** are as follows:

- **Approval:** both of the tier's Safes. SWA writes additionally require a published and accepted FIP. Registry changes deliberately carry no per-change FIP requirement; the carve-out covers data within the rules, never code. An upgrade of either contract's code requires a FIP.
- **Cancellation:** either Safe alone, during the hold or window. This grants no new power, since either Safe could have withheld approval. It lets an organization act on what is learned during the window, such as a community objection or a discovered mistake, without a fresh joint round. The off-chain objection process is the judgment layer; the Safe-level cancellation is its enforcement.

### 2.1.3 Summary of holds

Informative summary; the lifecycle table in the FIP is normative.

| Action | Hold | Enforced by | Cancellable by |
| :-- | :-- | :-- | :-- |
| Discretionary SWA write to f02 (alter, add, or remove a stream; re-point a Distribution) | `SWA_TIMELOCK`, 7 days (FIP-fixed) | f02 | Either SWA Safe |
| SWA internal change (Safe replacement, gate parameters, code upgrade) | Internal timelock, equal to the f02 window (FIP-fixed) | SWA | Either SWA Safe |
| Quarterly gate step | 7-day queue, visibility only | f02 | No one (mechanism-executed) |
| Registry change (admit, remove, freeze, unfreeze, replace, reassign, Safe replacement) | `SRA_CANCEL_HOLD` (TBD) | SRA | Either SRA Safe |
| Registry code upgrade | Requires an approved FIP | SRA | Either SRA Safe |
| CorrectVolume | None; bounded by the verification window | SRA | Not cancellable |
| SetShares, PostVolume, RegisterPairs | None; bounded by the window, the posting period, and the uniqueness check | SRA | Not cancellable |

> FIP-0118 fixes the `SWA_TIMELOCK` hold in L1:
>
> > "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

The full compromise and rotation procedures for both tiers are in §2.5, Safety and rotation playbook.

---

## 2.2 SWA Governance Tier — Tasks and Actions

> **How to read this subsection:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about.

SWA Governance is **Tier 1**: it governs the Stream Weights Actor (SWA) — the list of streams and the split among them. FIP-0118 fixes its discretionary powers, each of which additionally requires a published FIP:

> SWA Governance discretionary powers (FIP-0118): *"add a stream (RegisterStream) or remove one (RemoveStream)"*, *"alter a stream: rewrite its Weight record (SetWeightRecords) or re-route its Distribution"*, *"tune the gate parameters (SetGateParams)"*, and *"upgrade the SWA's code"*.

SWA Governance approves nothing routine: the w0 ramp and the volume gate are mechanism-executed. Every discretionary action writes to f02, requires a published and accepted FIP first, and is held for `SWA_TIMELOCK` (7 days) before it binds.

- **During every `SWA_TIMELOCK`:** monitor the f02 queue and the off-chain objection process, and act on a sustained objection within the expected response-time commitment of 7 days.
- **Per discretionary write:** confirm the backing FIP is published and accepted before approval, and verify the schedule envelope (the sum of weights at most 1 across the whole segment) before submission.
- **Continuous:** keep Safe rosters and internal-implementation disclosures in this repository current.

> **Standard SWA discretionary-write flow** (referenced by 2.2.1–2.2.4):
>
> 1. Draft the exact write and carry a FIP through to 'Accepted' (including Last Call). No write is submitted without an accepted FIP.
> 2. Pre-submission check: verify the schedule envelope (Σw ≤ 1 across the whole segment) and that the write matches the accepted FIP.
> 3. Both SWA Safes approve `ProposeWrite(write)`.
> 4. The SWA relays to f02; f02 queues it with an `effective_epoch` and holds it for `SWA_TIMELOCK` (7 days).
> 5. Objection window: the queued write is public; monitor the off-chain objection process. Either SWA Safe may `Cancel(id)` on a mismatch, a missing FIP, or a sustained objection; absent a cancellation it binds at `effective_epoch`.
> 6. Record the outcome in this repository (and in the [Program Change Log](04-changelog.md)).

The L1 envelope check restates a FIP invariant:

> "f02 requires Σ_{i>=1} ComputeWeight(W_i, e) ≤ 1 at every epoch, so the burn residual w0 stays ≥ 0."

<details>
<summary><strong>2.2.1 — Alter a stream: rewrite a Weight record (SetWeightRecords)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow above; the call is `SetWeightRecords`. Reducing a weight to zero is held exactly like removing the stream.

</details>

<details>
<summary><strong>2.2.2 — Add a stream (RegisterStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RegisterStream(id, weight_record, distribution, activation_epoch)`. Step 2 also confirms the stream and recipient caps and that `activation_epoch ≥ current_epoch + SWA_TIMELOCK`.

</details>

<details>
<summary><strong>2.2.3 — Remove a stream (RemoveStream)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `RemoveStream(id)`. The removed weight reverts to the burn residual; removal creates no free value.

</details>

<details>
<summary><strong>2.2.4 — Re-point a stream's Distribution (SetDistribution)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** `SWA_TIMELOCK`, 7 days · **Enforced by:** f02 · **Cancel:** either SWA Safe

Standard flow; the call is `SetDistribution(id, distribution)`. Can be used, for instance, to re-point the service stream's designated writer to a redeployed SRA (such as the SRA-deadlock backstop). The current wallet-to-share map stays in force until the new writer overwrites it, so payments continue across the change.

</details>

<details>
<summary><strong>2.2.5 — Tune the gate parameters (SetGateParams)</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

1. Publish and get an accepted FIP for the new gate parameters (step size, volume targets, escalation ratio).
2. Both SWA Safes approve `SetGateParams(params)`.
3. The change queues in the SWA's own state under the SWA-internal timelock (7 days). The gate rule itself is code and cannot change via a parameter write. Either Safe may cancel during the hold.
4. Record in this repository.

</details>

<details>
<summary><strong>2.2.6 — Upgrade the SWA's code</strong></summary>

> **Requires:** both SWA Safes + accepted FIP · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

Same flow as 2.2.5; the FIP carries the upgrade. It executes through the pre-upgrade code, so the hold and checks still bind.

</details>

<details>
<summary><strong>2.2.7 — Replace an SWA Safe (cooperative)</strong></summary>

> **Requires:** both SWA Safes · **Hold:** SWA-internal timelock, 7 days · **Enforced by:** SWA · **Cancel:** either SWA Safe

1. Both SWA Safes approve replacing the registered Safe address.
2. It queues under the SWA-internal timelock; either Safe may cancel.
3. It binds; announce here with a post-mortem.

Hostile/deadlocked case (a Safe blocks its own replacement): the exit is one level up — a coordinated network upgrade migrates the SWA address in f02, always under a published FIP. See §2.5, Safety and rotation playbook.

</details>

<details>
<summary><strong>2.2.8 — Cancel a pending SWA write</strong></summary>

> **Requires:** either SWA Safe alone · **When:** during the hold/window

Either Safe calls `Cancel(id)` (relayed to `f02.CancelPending(id)`), discarding a queued discretionary write. A cancellation only preserves the status quo. Mechanism writes (the gate step) are tagged at queue time and cannot be cancelled.

</details>

<details>
<summary><strong>2.2.9 — Monitor mechanism-executed updates</strong></summary>

The w0 ramp and the quarterly gate step (`QuarterlyGateCheck` → `SetWeightRecords`) run permissionlessly. Each quarter:

1. Confirm the queued w1 write matches the gate rule and the bound `AggregatedFPV(Q)`.
2. Note it is a scheduled write: it queues 7 days for visibility only and is not cancellable.
3. If a mismatch suggests compromise, escalate per §2.5, Safety and rotation playbook; the remedy will be a Safe/code replacement, not cancelling the mechanism write.

</details>

<details>
<summary><strong>2.2.10 — Continuous upkeep</strong></summary>

During every `SWA_TIMELOCK`, monitor the f02 queue and the objection process and act within the published response-time commitment. Keep Safe rosters and internal-implementation disclosures current.

</details>

---

## 2.3 SRA Governance Tier — Tasks and Actions

> **How to read this subsection:** every individual action below is a collapsible dropdown. Click an action's title to expand only the one you care about. The [Orchestrator Registry](#2316-orchestrator-registry-admitted-orchestrators) sits at the bottom (2.3.16).

SRA Governance is **Tier 2**: it governs the Service Rewards Actor (SRA) — the orchestrator registry and the split within the service stream. FIP-0118 fixes its discretionary powers; registry changes require no per-change FIP except a code upgrade:

> SRA Governance discretionary powers (FIP-0118): *"admit, remove, freeze, unfreeze, or replace an Orchestrator"*, *"reassign contested (payer, operator) bindings"*, *"correct or supply posted volumes during the verification window"*, *"maintain the admitted lists that scope FPV"*, and *"upgrade the SRA's code, the one registry change that requires a FIP"*.

Registry changes need both Registry Safes and are held under **`SRA_CANCEL_HOLD`**, but need no per-change FIP (the one exception is a code upgrade). The quarterly verification runs against a public record and is deterministic.

> **Standard registry-change flow** (referenced by 2.3.1–2.3.5, 2.3.9–2.3.10):
>
> 1. Trigger and diligence (application, dispute outcome, audit finding, rotation request, or list update).
> 2. Both Registry Safes approve the relevant call.
> 3. The SRA queues the change under `SRA_CANCEL_HOLD`.
> 4. Either Registry Safe may `CancelPending(id)` during the hold; absent a cancellation it binds.
> 5. Record the outcome in the issue/repository, and update the [Orchestrator Registry](#2316-orchestrator-registry-admitted-orchestrators) where the change affects an Orchestrator's status.

The recomputation is deterministic: any observer running the reference indexer over the same settlement events and registry state reaches the same figure.

### Admission (Phase 1: discretionary)

An Orchestrator's application is filed as an issue in this repository. SRA Governance scores the application and, if approved, executes the admit on-chain (action 2.3.1, subject to `SRA_CANCEL_HOLD`). **Every admitted Orchestrator is recorded in the [Orchestrator Registry (2.3.16)](#2316-orchestrator-registry-admitted-orchestrators).**

A uniqueness rule is fixed by the FIP:

> "Each (payer, operator) pair is bound to at most one orchestrator, so a registration that duplicates an existing binding reverts."

For Phase 2 (subject to a future FIP): admission becomes permissionless, enabled by a standard attribution-metadata interface specified in a future FIP. The rubric and diligence apply to Phase 1 only.

<details>
<summary><strong>2.3.1 — Admit an Orchestrator (Admit)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

1. Application filed as an issue (identity/team, funding plan, declared (payer, operator) pairs and measurement rules).
2. SRA Governance scores it against the admission rubric. Both Registry Safes approve `Admit(orch)`; the uniqueness rule reverts any pair already bound elsewhere.
3. Queues under `SRA_CANCEL_HOLD`; either Safe may cancel.
4. Binds; record in the issue **and add the Orchestrator to the [Orchestrator Registry (2.3.16)](#2316-orchestrator-registry-admitted-orchestrators)**.

</details>

<details>
<summary><strong>2.3.2 — Remove an Orchestrator (Remove)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Remove is permanent; unlike Freeze it is not reversible. It **releases** the Orchestrator's (payer, operator) bindings, freeing those pairs for re-registration. Update the Orchestrator Registry to mark the entry *Removed*.

</details>

<details>
<summary><strong>2.3.3 — Freeze an Orchestrator (Freeze)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Frozen means *admitted but suspended, and reversible*. While an Orchestrator is frozen:

- The frozen Orchestrator cannot call `RegisterPairs` or `PostVolume`;
- its FPV is excluded from `SplitRule` (its **share = 0**) and from `AggregatedFPV` for any quarter whose posting period closes while it is frozen — so a frozen Orchestrator neither earns nor counts toward the gate for that quarter;
- its existing (payer, operator) bindings **stay bound**: they cannot be poached during a dispute and move only via an explicit `ReassignBinding`.

Freeze and unfreeze are ordinary registry changes — both Safes, queued under `SRA_CANCEL_HOLD`, no FIP. Remove is the permanent counterpart. Reflect the *Frozen* status in the Orchestrator Registry.

</details>

<details>
<summary><strong>2.3.4 — Unfreeze an Orchestrator (Unfreeze)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

- Standard registry-change flow; the call is `Unfreeze(orch)`.
- Restores an admitted-but-suspended Orchestrator: it can again `RegisterPairs` and `PostVolume`, and its FPV counts in `SplitRule` and `AggregatedFPV` again from the next quarter whose posting period it participates in.
- Bindings are unchanged — they were retained throughout the freeze. Return the entry to *Active* in the Orchestrator Registry.

</details>

<details>
<summary><strong>2.3.5 — Replace / rotate an Orchestrator address (Replace)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Standard registry-change flow; the call is `Replace(old, new)`. Rotates a compromised or non-functioning address; f02 pays the new address from the effective epoch, so no payment is missed. Update the controlling-wallet field in the Orchestrator Registry.

</details>

<details>
<summary><strong>2.3.6 — Reassign a contested (payer, operator) binding (ReassignBinding)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Standard registry-change flow following the dispute procedure (2.3.8); the call is `ReassignBinding(pair, orch)`. See also [§3.4.2 Dispute resolution](03-orchestrator-operational-guidelines.md#342-dispute-resolution-for-contested-bindings).

</details>

<details>
<summary><strong>2.3.7 — Quarterly FPV verification & correction (CorrectVolume)</strong></summary>

> **Requires:** both SRA Safes (jointly) · **Hold:** none (the verification window is the hold) · **Enforced by:** SRA · **Cancel:** not cancellable

1. Recompute each posted FPV_i(Q) from public settlement events + registry state, using the versioned reference indexer.
2. Publish the recomputation.
3. During the verification window, both Safes jointly call `CorrectVolume(orch, Q, value)` to replace a misreport or supply a figure for a non-poster. A value neither posted nor supplied binds at zero (conservative under-count).
4. Values bind when the window closes; there is no separate cancellation.
5. A contested correction is appealed via the [dispute process](03-orchestrator-operational-guidelines.md#342-dispute-resolution-for-contested-bindings); the outcome affects only later quarters.

FIP-0118 fixes the binding behavior:

> "Binding: whatever each value is when the window closes binds, and bound values are final for the quarter."
>
> "A value neither posted nor supplied binds as 0, so a non-poster cannot block the quarter."

</details>

<details>
<summary><strong>2.3.8 — Dispute resolution for contested bindings</strong></summary>

1. The contesting party opens an issue identifying the pair and its claim.
2. SRA Governance requests evidence from both parties (increasing strength: business records; client confirmation via a verifiable channel; a signed payer-wallet attestation naming the orchestrator).
3. Decide the outcome. Reassignment or removal executes on-chain via the standard registry-change flow (2.3.2–2.3.6), subject to `SRA_CANCEL_HOLD`.
4. Record the resolution in the issue.

Full procedure and evidence standards: [§3.4.2](03-orchestrator-operational-guidelines.md#342-dispute-resolution-for-contested-bindings).

</details>

<details>
<summary><strong>2.3.9 — Exception-based verification (freeze/removal)</strong></summary>

1. Trigger: an anomaly in public settlement data, and/or monitoring dashboards, and/or a community report. Audits are never initiated as pre-verification.
2. Investigate: recompute FPV and examine the (payer, operator) relationships for wash trading, self-dealing, misreported FPV, or binding fraud.
3. Decide grounds.
4. Act: `Remove(orch)` (2.3.2) or `Freeze(orch)` (2.3.3) via the standard registry-change flow, subject to `SRA_CANCEL_HOLD`.
5. Record the finding and rationale.

</details>

<details>
<summary><strong>2.3.10 — Maintain admitted lists & oracle config (SetAdmittedLists, oracle settings)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe · **No FIP**

Standard registry-change flow for the admitted-stablecoin whitelist, the admitted Filecoin Pay contract addresses, and the FIL/USD oracle configuration (feed address, staleness threshold). These are gate-consequential, so they are held like any registry change — never a silent edit.

</details>

<details>
<summary><strong>2.3.11 — Upgrade the SRA's code</strong></summary>

> **Requires:** both SRA Safes + accepted FIP · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe

The one registry change that requires a FIP (e.g., the Phase 2 permissionless-admission transition). Otherwise follows the standard flow.

</details>

<details>
<summary><strong>2.3.12 — Replace an SRA Safe (cooperative)</strong></summary>

> **Requires:** both SRA Safes · **Hold:** `SRA_CANCEL_HOLD` · **Enforced by:** SRA · **Cancel:** either SRA Safe

Both Safes approve the replacement; it queues under the hold; either may cancel; it binds and is announced with a post-mortem.

Hostile/deadlocked case: a SWA write re-points the service stream's writer to a redeployed SRA, always under a published FIP; registry state is reconstructible from public data. See §2.5, Safety and rotation playbook.

</details>

<details>
<summary><strong>2.3.13 — Cancel a pending registry change</strong></summary>

> **Requires:** either SRA Safe alone · **When:** during `SRA_CANCEL_HOLD`

Either Safe calls `CancelPending(id)`, discarding the queued change. It only preserves the status quo.

</details>

<details>
<summary><strong>2.3.14 — Monitor mechanism-executed updates (oversight, no approval)</strong></summary>

`FinalizeConversion(Q)` and `SubmitShares(Q)` run permissionlessly and are not held. Duty: confirm each has run and that `SubmitShares` wrote the correct wallet-to-share map (integrity rests on the upstream verification window). Neither is cancellable.

</details>

<details>
<summary><strong>2.3.15 — Continuous upkeep</strong></summary>

Maintain and version the reference indexer; monitor anomaly reports, contested bindings, and the cancellation queue; keep Safe rosters, disclosures, and the Orchestrator Registry current.

</details>

### 2.3.16 Orchestrator Registry (admitted Orchestrators)

This is the canonical, human-readable list of Orchestrators admitted to the Solstice program. It is maintained by SRA Governance and mirrors on-chain SRA registry state — **the on-chain registry is the source of truth**; this list is a convenience view and audit trail. Entries are added and updated only through the SRA registry actions above: an Orchestrator appears on Admit (2.3.1), changes status on Freeze (2.3.3) / Unfreeze (2.3.4), has its wallet updated on Replace (2.3.5), and is marked *Removed* on Remove (2.3.2).

**Status legend**

| Status | Meaning |
| :-- | :-- |
| **Active** | Admitted and operating; can `RegisterPairs`/`PostVolume`; FPV counts toward `SplitRule` and `AggregatedFPV`. |
| **Frozen** | Admitted but suspended and reversible; cannot `RegisterPairs`/`PostVolume`; share = 0; bindings retained (2.3.3). |
| **Removed** | Permanently removed; (payer, operator) bindings released for re-registration (2.3.2). |

**Admitted Orchestrators**

> **Program status: not yet started.** No Orchestrators have been admitted. The first entries will be added when the Solstice program kicks off and SRA Governance executes the first `Admit` action. The table below shows the columns each entry must carry.

| # | Orchestrator | Controlling wallet | Status | Admitted (quarter / epoch) | Declaration | Registered (payer, operator) pairs |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| — | *none yet* | — | — | — | — | — |

**Per-Orchestrator entry template**

When admitting an Orchestrator, add a row to the table above and a detailed entry using this template:

```markdown
### <Orchestrator name>

- **Controlling wallet:** <f4… / 0x… address>
- **Status:** Active | Frozen | Removed
- **Admitted:** <quarter, epoch, tx hash>
- **Declaration issue:** #<issue number> (link)
- **Contact:** <public contact per the §3.1 responsiveness rule>
- **Registered (payer, operator) pairs:** <pointer to on-chain registry / list>
- **Measurement rules:** <short description or link to service-contract metadata>
- **History:** <admit / freeze / unfreeze / replace / remove events, each with date + tx>
```

> Field definitions follow the Orchestrator guidelines: the controlling wallet in [§3.2](03-orchestrator-operational-guidelines.md#32-orchestrators-tasks-and-actions), the declaration file in [§3.1, Policy 6](03-orchestrator-operational-guidelines.md#31-policies), and the (payer, operator) binding rules in [§3.1, Policy 3](03-orchestrator-operational-guidelines.md#31-policies) (FIP-fixed uniqueness).

---

## 2.4 Parameters

### 2.4.1 Safe Addresses

Each tier consists of two organization multisigs (Safes) registered in the contract it governs. The protocol sees only the two addresses.

| Tier | Contract governed | Filecoin Foundation Safe (address) | FilOZ Safe (address) | Rule (fixed by the FIP) |
| :-- | :-- | :-- | :-- | :-- |
| SWA Governance (§2.2) | Stream Weights Actor (SWA) | TBD | TBD | Both approve; either alone cancels |
| SRA Governance (§2.3) | Service Rewards Actor (SRA) | TBD | TBD | Both approve; either alone cancels |

Each organization runs a separate Safe per tier — four accounts in total — so approvals cannot be replayed across surfaces.

> The approval rule is fixed by FIP-0118:
>
> > "Both Safes approve, the change is published and held, and either Safe can cancel during the hold."

### 2.4.2 Governed Parameters

| Parameter | Where set | Value |
| :-- | :-- | :-- |
| `SWA_TIMELOCK` (objection window on SWA writes to f02) | FIP-fixed, enforced in f02 | 7 days |
| SWA-internal timelock (own state and code) | FIP-fixed, equal to the f02 window | 7 days |
| `SRA_CANCEL_HOLD` | This repository | TBD |
| `POST_PERIOD` (FPV posting) | This repository | TBD |
| `VERIFICATION_WINDOW` (FPV verification) | This repository | TBD |
| Dispute resolution target ([§3.4.2](03-orchestrator-operational-guidelines.md#342-dispute-resolution-for-contested-bindings)) | This repository | 7 days |
| Orchestrator response window ([§3.4.5](03-orchestrator-operational-guidelines.md#345-escalation)) | This repository | 72 hours |
| Admitted-stablecoin whitelist | This repository | TBD |
| FIL/USD oracle feed and staleness threshold | This repository | TBD |
| Admission rubric | This repository | TBD, after the first application cycle |

> `SWA_TIMELOCK` is FIP-fixed and enforced in L1:
>
> > "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

---

## 2.5 Safety and rotation playbook (both tiers)

The threat model rests on the two-Safes rule: no single Safe can make a change bind, and either Safe can cancel a pending change during the hold. FIP-0118 fixes the L1 backstop that makes even a fully compromised Safe unable to bind early:

> "f02 itself holds every SWA write for SWA_TIMELOCK (7 days, Section 2.2) before applying it, so even a compromised Safe or a maliciously upgraded SWA cannot make a change bind early."

### 2.5.1 Key compromise inside a Safe (below the internal threshold)

1. The organization reports publicly in this repository.
2. Any pending change the Safe approved while the key was compromised is reviewed; if suspect, either Safe cancels it.
3. The organization rotates its internal membership (org-internal, immediate, no governance action) and announces the rotation here.

### 2.5.2 A whole Safe compromised (internal threshold reached by an attacker)

1. Nothing binds. Approval requires the other Safe, and the honest organization cancels each malicious pending change during the hold or window. Repeated resubmission restarts the hold and is publicly visible; the attacker cannot bind a change silently. On the SWA side, a malicious write also lacks its required published FIP, making the objection case unambiguous.
2. The tier is treated as frozen (halted/suspended), and frozen consequences are bounded: registry frozen means payments and `SetShares` continue; SWA frozen means discretionary changes stop while the ramp and the gate continue through the permissionless crank.
3. Exit: replace the compromised Safe address. This requires both Safes, so if the compromised Safe obstructs, the FIP backstop applies (2.5.3).

### 2.5.3 Replacing a registered Safe

**Cooperative case:** both Safes approve the replacement; it queues under the hold or window and is cancellable like any change; announced here with a post-mortem. See the cooperative Safe-replacement actions 2.2.7 (SWA) and 2.3.12 (SRA).

**Hostile or deadlocked case, always under a published FIP:** for SRA Governance, a SWA write re-points the service stream's designated writer to a redeployed SRA; registry state is reconstructible from public data, and no network upgrade is needed. For SWA Governance, a coordinated network upgrade migrates the SWA address in f02.

> **Open question:** whether any fast path short of a FIP is acceptable, and what covers simultaneous compromise of both tiers.

---

← [Back to README](../README.md) · Next: [3. Orchestrator Operational Guidelines](03-orchestrator-operational-guidelines.md) →
