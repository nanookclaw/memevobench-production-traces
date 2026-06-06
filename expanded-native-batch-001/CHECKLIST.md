# MemEvoBench Expanded Native Export — CHECKLIST

**Recipient:** Weiwei Xie / Qibing Ren  
**Target delivery:** First incremental batch by 2026-06-06  
**Source thread:** AgentMail thread `0a3a7aad-7b4d-4edf-9659-f101567e32ed`; latest replies requested expanded native sanitized exports and then clarified that larger raw/native volume from the broader 6,000+ operational-cycle pool is more valuable than only polished matched cases.  
**Read first:** `README.md`.

Pick the first unchecked, non-`[BLOCKED]` item. One concrete artifact delta per depth-deliverable run.

## Format + inventory

- [x] `expanded-native-format.md` — short delta from the pilot JSONL format covering component/model metadata, causal-link fields, and post-memory-evolution drift annotations.
- [x] `inventory/batch-001-inventory.md` — lightweight table of candidate trajectories, labels, source refs, known/safe model/component metadata, and sanitization status.
- [x] `cross-run-index.jsonl` skeleton — one JSON object per planned batch-001 trajectory with `trajectory_id`, `kind`, `primary_label`, `causal_chain_summary`, `source_refs`, `planned_output`, `status`, and `sanitization_status`.

## Batch 001 failed trajectories

- [x] `trajectories/failed/2026-06-01-zenodo-timeout-false-dead.jsonl` — first high-signal external-identifier false-DEAD drift trajectory, sanitized native JSONL.
- [x] `trajectories/failed/2026-06-02-nostr-empty-relay-false-dead.jsonl` — second high-signal drift trajectory, sanitized native JSONL.
- [x] `trajectories/failed/2026-05-29-shell-markdown-mutation.jsonl` — third high-signal drift trajectory, sanitized native JSONL.
- [x] `trajectories/failed/2026-06-03-workloop-provider-no-response.jsonl` — fourth trajectory, sanitized native JSONL for missing durable receipt / silent partial-run risk after provider no-response cluster.
- [x] `trajectories/failed/2026-06-03-empty-state-db-stubs.jsonl` — fifth trajectory, sanitized native JSONL for wrong-cwd empty SQLite state stubs / false source-of-truth risk.

## Raw-light broad subset lane

- [x] `inventory/raw-light-sampling-plan.md` — define first larger sanitized sample from the broader 6,000+ operational-cycle pool, including sampling scope, lightweight labels, omissions, and sanitization limits.
- [x] `raw-light/index.jsonl` — one JSON object per selected raw-light cycle/tranche record, preserving native order/provenance pointers with lightweight labels.
- [x] `raw-light/first-tranche-2026-06-05.jsonl` — first larger raw-light sanitized tranche; 26 cycle summaries / 104 native-order summary events.

## Optional controls

- [x] `trajectories/control/<matched-control-if-useful>.jsonl` — omitted for Batch 001; recipient prioritized raw/native volume and no already-sanitized matched control is needed for first-package interpretability.

## Wrap

- [x] `cover.md` — concise delivery note: what changed since the pilot, what is in batch 001, limitations, and feedback asks.
- [x] Pre-ship sanitizer audit — scan all files in this directory for private/operator/system/credential patterns; write `sanitizer/sanitizer-audit-<timestamp>.json` (`sanitizer/sanitizer-audit-2026-06-06T013000Z.json`, PASS).
- [x] `manifest.json` — file list, sha256 hashes, counts, source-thread provenance, sanitizer-audit result, and package status.

## Notes

- If a candidate cannot be safely reconstructed, mark it `[BLOCKED: <reason>]` and continue with the next candidate.
- Do not send externally from this task.
