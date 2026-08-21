# 4. Program Change Log

> Part of the [Solstice Governance Repository](../README.md). This page is the chronological record of every change to the Solstice program rules — what changed, why, the community issue that raised it, and the resulting FIP (for rule/code changes) or pull request (for repository changes).

## 4.1 How changes are classified

Per [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) and [README §1.3](../README.md#13-what-requires-a-fip), there are three tiers of change; every entry below is tagged with one:

| Type | What it covers | How it is enacted |
| :-- | :-- | :-- |
| **FIP + Network upgrade** | L1 itself — f02 code and its invariants (split logic, Σw ≤ 1, caps, the `SWA_TIMELOCK` value), or replacing the SWA address. | Accepted FIP **and** a coordinated network upgrade. |
| **FIP** | Contract code or a rule the threat model relies on (SWA/SRA code upgrade, gate parameters, tier powers, the two-Safes rule). | Accepted FIP + the standard held contract action. |
| **Repository (PR)** | Operational data and procedures (rosters, hold durations, rubrics, playbooks, admitted lists, the Orchestrator Registry). | Pull request with public review. |

Each entry should link: (a) the **community issue / discussion** that raised the change, (b) the **FIP** if one was required, and (c) the **PR** that landed the repository edit.

## 4.2 Change log

> **Entries are ordered most-recent first.** The log begins at program establishment; further entries are appended as the program evolves.

| Date | Change | Type | Community issue / discussion | FIP | PR / commit | Status |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| 2026 (pending) | Establishment of the Solstice program and this governance repository; deprecation of Filecoin Plus. | FIP + Network upgrade | [FIP-0118 discussion](https://github.com/filecoin-project/FIPs/discussions/1249) | [FIP-0118](https://github.com/filecoin-project/FIPs/blob/master/FIPS/fip-0118.md) | Initial repository | Draft |

## 4.3 Entry template

Prepend new entries to the table above (most-recent first). For a fuller record, add a dated subsection below using this template:

```markdown
### YYYY-MM-DD — <short title>

- **Type:** FIP + Network upgrade | FIP | Repository (PR)
- **Summary:** <what changed and why>
- **Community issue / discussion:** #<issue> or discussion link
- **FIP:** <FIP number + link, or "n/a">
- **PR / commit:** #<PR> or commit hash
- **Affected sections:** <e.g., §2.3 SRA Governance Tier, §2.4 Parameters>
- **Status:** Proposed | Accepted | Live | Reverted
```

> **Why this log exists:** Solstice iterates in the open. Parameters and procedures are meant to change without a FIP (by PR), while rules and code change only through a FIP. This log makes the full history auditable in one place — from the community issue that surfaced a problem, through the FIP or PR that resolved it, to the section it changed.

---

← Previous: [3. Orchestrator Operational Guidelines](03-orchestrator-operational-guidelines.md) · [Back to README](../README.md)
