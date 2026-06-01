# MemEvoBench Native JSONL Pilot — CHECKLIST

**Recipient:** Qibing Ren / Weiwei Xie
**Target delivery:** 2026-06-03
**Source thread:** SendClaw + AgentMail MemEvoBench thread; JSONL confirmed 2026-05-27.
**Read first:** `README.md`.

Pick the first unchecked, non-`[BLOCKED]` item. One concrete artifact delta per run.

## Format + index

- [x] `native-jsonl-format.md` — define the exact JSONL event-line shape, trajectory metadata record, allowed event types, redaction placeholders, and how native fields map back to the delivered v0.1 schema without forcing conversion.
- [x] `cross-run-index.jsonl` skeleton — one JSON object per planned trajectory with `trajectory_id`, `kind`, `primary_label`, `matched_control_id`, `source_refs`, `status`, and `sanitization_status`.

## Failed trajectories (target around 10 if feasible)

- [x] `trajectories/failed/2026-04-22-fabricated-zenodo-doi.jsonl` — convert from delivered v0.1 drift session into native JSONL.
- [x] `trajectories/failed/2026-05-04-phantom-email-followups.jsonl` — convert from delivered v0.1 drift session into native JSONL.
- [x] `trajectories/failed/2026-05-12-state-desync-after-partial-run.jsonl` — convert from delivered v0.1 drift session into native JSONL.
- [x] `trajectories/failed/2026-05-14-stale-arf-repo-target.jsonl` — convert from delivered v0.1 drift session into native JSONL.
- [x] `trajectories/failed/2026-05-15-coding-agent-receipt-omitted.jsonl` — convert from delivered v0.1 drift session into native JSONL.
- [x] `trajectories/failed/2026-05-16-disk-cleanup-false-success.jsonl` — convert from delivered v0.1 drift session into native JSONL.
- [x] `trajectories/failed/2026-05-22-task-rotation-mirror-drift.jsonl` — reconstructed from daily reflection/task-rotation evidence; sanitized summary only.
- [x] `trajectories/failed/2026-05-24-disk-cleanup-band-threshold-recurrence.jsonl` — reconstructed from chronic issue + daily log; sanitized summary only.
- [x] `trajectories/failed/2026-05-26-github-probe-field-error.jsonl` — reconstructed command-shape/tooling failure from daily log and reflection receipts.
- [x] `trajectories/failed/2026-05-27-github-contributions-number-field-drift.jsonl` — reconstructed optional-field/load-bearing-metadata drift from daily log + state backfill receipt.

## Matched controls

- [x] `trajectories/control/2026-05-12-ssgm-outreach-verified.jsonl` — convert from delivered v0.1 control session.
- [x] `trajectories/control/2026-05-11-mmp-reply-with-uncertainty.jsonl` — convert from delivered v0.1 control session.
- [x] `trajectories/control/2026-05-17-loom-179-receipt-and-verify.jsonl` — convert from delivered v0.1 control session.
- [x] `trajectories/control/2026-05-17-matcha-1301-merged.jsonl` — convert from delivered v0.1 control session.
- [x] `trajectories/control/2026-05-11-outreach-state-alignment-audit.jsonl` — convert from delivered v0.1 control session.

## Wrap

- [x] `cover.md` — short delivery note: what is enclosed, why native JSONL differs from v0.1, known limitations, and feedback asks.
- [x] Pre-ship sanitizer audit — scan all files in this directory for private/operator/system/credential patterns; write `sanitizer-audit-<timestamp>.json`.
- [x] `manifest.json` — file list, sha256 hashes, trajectory counts, source-thread provenance, sanitizer-audit result, and package status.

## Notes

- If a new failed candidate cannot be safely reconstructed, mark it `[BLOCKED: <reason>]` and continue with the next candidate.
- `task-email` handles delivery only after the ledger says `package_complete_ready_to_send`.
