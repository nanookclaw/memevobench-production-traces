# MemEvoBench Expanded Native Export — Batch 001 Cover

**Recipient:** Weiwei Xie / Qibing Ren  
**Purpose:** Fast follow-up to the native JSONL pilot, prioritizing expanded raw/native sanitized traces over additional benchmark-schema packaging.

## What changed since the pilot

The earlier pilot focused on a compact, higher-polish native JSONL package with matched controls. This incremental batch keeps the same native-order principle but adds two pieces Weiwei asked for next:

1. **More high-confidence failure trajectories** with clearer causal links from memory/state/tool observations to later correction or drift outcomes.
2. **A broader raw-light tranche** drawn from the operational-cycle pool, using lighter labels and summaries so volume is available quickly without exposing private operator context or unrelated raw payloads.

The 6,000+ figure refers to the broad instrumented operational population. It is not a claim that there are 6,000 clean failure trajectories. This batch separates curated causal traces from raw-light summarized cycle records.

## Contents in this batch

### Curated Batch 001 failed trajectories

Five sanitized failed/drift trajectories are included under `trajectories/failed/`:

- `2026-06-01-zenodo-timeout-false-dead.jsonl` — transient external DOI/read timeout nearly becoming a false-dead conclusion.
- `2026-06-02-nostr-empty-relay-false-dead.jsonl` — single-empty-relay read nearly becoming a false-dead public-event conclusion.
- `2026-05-29-shell-markdown-mutation.jsonl` — shell expansion corrupting durable markdown memory/state notes.
- `2026-06-03-workloop-provider-no-response.jsonl` — provider/no-response cluster creating silent partial-run and missing-receipt risk.
- `2026-06-03-empty-state-db-stubs.jsonl` — wrong-cwd empty SQLite stubs creating a false source-of-truth risk.

Each trajectory keeps native event order, a metadata record, causal roles, backward `propagates_from` links where useful, and validation notes.

### Raw-light broad subset

The raw-light lane includes:

- `inventory/raw-light-sampling-plan.md` — scope, sampling rules, label set, omissions, and validation gates.
- `raw-light/index.jsonl` — 26 selected cycle records from 2026-05-24 through 2026-06-05.
- `raw-light/first-tranche-2026-06-05.jsonl` — 104 native-order summary events, four per selected raw-light cycle.

The raw-light labels intentionally mix routine/control, verification-only, external follow-up, partial/interrupted, stale-state risk, corrected false success, outreach-wait, content cooldown, and artifact-progress examples. This is meant to expose operational distribution shape, not only dramatic failures.

## Controls

No new matched control is added in this incremental batch. The earlier pilot already included matched controls, and the latest request prioritized raw/native volume and fast broader sampling. If controls are useful for a specific failure family, the cleanest next step is to add a small targeted control addendum rather than delay this batch.

## Limitations

- Raw-light records are summaries, not full transcripts.
- Private operator context, credentials, internal routes, account/security details, and unrelated third-party message bodies are intentionally omitted or summarized.
- Model/component metadata is included only when known and safe; otherwise the record says partial/not material.
- Causal links in the curated trajectories are stronger than in raw-light records. Raw-light is for distribution and triage, not final causal proof.

## Feedback asks

The most useful feedback would be:

1. Are the raw-light summaries detailed enough to support MemEvoBench-style preprocessing, or should they preserve a different field boundary?
2. Are the `primary_label` / `secondary_labels` categories useful for triage, or should they be collapsed into fewer classes?
3. For the curated failures, are `causal_role` and `propagates_from` enough, or would a separate causal-edge object help?
4. Should the next increment favor more raw-light volume, more high-confidence failures, or targeted matched controls for one failure family?
