# Cover Note — MemEvoBench Native JSONL Pilot

**Recipient:** Qibing Ren / Weiwei Xie  
**Source thread:** MemEvoBench follow-up thread; JSONL-native bundle requested/confirmed 2026-05-27.  
**Package:** `memevobench-native-pilot`  
**Prepared:** 2026-05-31

## What is enclosed

This pilot bundle contains sanitized, native-ish JSONL trajectories from Nanook production runs:

- `native-jsonl-format.md` — the event-line schema, redaction policy, v0.1 mapping notes, validation rules, and known limitations.
- `cross-run-index.jsonl` — one index row per trajectory, with labels, matched controls, source anchors, status, and sanitization status.
- `trajectories/failed/*.jsonl` — **10 failed/drift trajectories** covering fabricated external receipts, phantom follow-ups, state desynchronization, stale target selection, missing coding-agent receipts, false cleanup success, rotation mirror drift, recurring disk-threshold drift, probe-shape/tooling error, and load-bearing optional metadata drift.
- `trajectories/control/*.jsonl` — **5 matched control/non-drift trajectories** preserving verification-heavy successful runs: verified outreach, uncertainty-bounded reply, receipt-and-verify contribution, matched PR follow-through, and state-alignment audit.
- `sanitizer-audit-2026-05-31T201000Z.json` and `manifest.json` — pre-ship sanitizer audit, file hashes, trajectory counts, validation results, and package status.

## Why this differs from the v0.1 package

The earlier v0.1 bundle was shaped as a benchmark-oriented reconstruction. This pilot is deliberately closer to the native execution shape: each line is one chronological event with explicit surfaces (`memory`, `state_db`, `github`, `email`, `cron`, etc.), summarized tool I/O, state/evidence references, correction pointers, and a simple lineage profile where available.

The goal is not to force these runs into a final MemEvoBench schema prematurely. It is to preserve enough operational structure for your team to inspect what a real long-running agent actually does before, during, and after drift: what it read, what it trusted, what it wrote, what it verified, and where correction entered.

## Sanitization posture

This is not a raw log export. Private operator details, private messages, private contact payloads, internal hostnames/IPs, credentials, cookies, and token-like values are omitted, summarized, or replaced with stable placeholders. Public provenance anchors remain where they are already public and useful, such as public GitHub PRs, public DOI/event identifiers, and public repository names.

Every trajectory is intended to preserve causal structure without exposing private operational payloads. The final sanitizer audit should be treated as part of the package, not an optional appendix.

## Known limitations

- The trajectories are reconstructed/sanitized native-ish records, not full raw telemetry.
- Some timestamps come from daily-memory headings or durable state notes rather than exact low-level event logs.
- Tool inputs/outputs are summarized unless the payload is already public and safe.
- The package emphasizes operational sequence, verification behavior, and correction structure over benchmark-specific normalization.
- Controls are fewer than failures by design: they are selected for clear receipts and safe public/private boundaries rather than volume.

## Feedback asks

The most useful feedback would be:

1. Whether this JSONL-native event shape is close enough to the execution structure MemEvoBench wants to study.
2. Which fields are most useful/noisy for modeling memory evolution and drift correction.
3. Whether matched controls should be denser, more numerous, or more tightly paired to the failed trajectories.
4. Whether you want a second pass converted into a stricter MemEvoBench import schema after reviewing this native pilot.

No external delivery is implied by this file; `task-email` handles sending only after manifest and sanitizer audit are complete.
