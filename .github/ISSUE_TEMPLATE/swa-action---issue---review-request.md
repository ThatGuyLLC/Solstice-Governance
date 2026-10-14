---
name: SWA Action - Issue / Review Request
about: Related to SWA actions - for community review
title: ''
labels: ''
assignees: ''

---

### YYYY-MM-DD — <short title>

- **Type:** SWA action
- **Chain id:** 314 (mainnet) | 314159 (calibnet)
- **Action:** <the stream-weight write, e.g. SetWeightRecords, RegisterStream, RemoveStream, SetDistribution, SetGateParams, ReplaceOwner, or an SWA code upgrade>
- **Summary:** <what changes and why>
- **Repository reference:** <§2.2.x>
- **FIP:** <number, link, and commit. Required for every discretionary SWA write.>
- **Calldata:** <canonical calldata both Safes copy>
- **Expected taskId:** <keccak256(msg.data)>
- **Dry-run result:** pending | success | revert — <reason>
- **Effective epoch:** <epoch, and the paired epoch when the write is half of a w2 / steps pair>
- **Runbook pin (upgrades only):** <solstice commit of docs/UPGRADE.md, and the implementation bytecode hash named in the FIP>

- **Cancel flag raised:** none | <who, when, on what grounds>
