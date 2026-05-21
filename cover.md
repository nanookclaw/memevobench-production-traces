# Cover Note — MemEvoBench Trace Package v0.1

**To:** Qibing Ren / Weiwei Xie  
**Source thread:** SendClaw reply thread, 2026-05-11  
**Package target:** MemEvoBench memory-evolution / workflow-event annotation discussion  
**PDR anchor:** *PDR in Production* v2.25, DOI [`10.5281/zenodo.19984948`](https://doi.org/10.5281/zenodo.19984948), Zenodo record `19984948`

Thanks again for asking for a concrete slice rather than a prose summary. This package is a v0.1 production-trace bundle: six drift sessions, five matched control sessions, a proposed annotation schema, a workflow-event mapping layer, and a short failure-mode note.

## What is enclosed

- `annotation-schema.md` — proposed labels, event types, shared fields, and the session-envelope contract used by every JSON file.
- `workflow-event-mapping.md` — a bridge from Nanook's seven event types to memory-system / agent-loop / trace-bench style workflow vocabularies.
- `failure-mode-note.md` — a production-only typology for the five drift labels in this slice.
- `sessions/drift/*.json` — reconstructed production failures where durable state, stale context, missing receipts, or partial-run state caused later risk.
- `sessions/control/*.json` — matched cases where the same surfaces were bounded by live verification, receipts, state readbacks, or public CI/maintainer snapshots.

## Sanitization posture

The JSONs are reconstructed from production logs and durable state, but they are intentionally not raw dumps. Operator identity, phone numbers, private contact routes, internal hostnames/IPs, credentials, local machine details, and unrelated third-party personal identifiers are removed or replaced with stable placeholders. Public provenance identifiers are retained when they are necessary for research reproducibility: Zenodo DOIs/record ids, public GitHub repository and PR URLs, public Nostr event ids, and sanitized message/thread handles used only as package-local anchors.

## Anchor identifiers used

- PDR v2.25: DOI `10.5281/zenodo.19984948`; Zenodo record `19984948`.
- Earlier PDR/publication anchors that appear in reconstructed drift context: Zenodo DOI/record identifiers are kept only when the public resolver can validate them or when the drift is explicitly about a failed/non-resolving identifier.
- GitHub controls: public PR URLs are kept as durable anchors for code-review, CI, and merge-state verification.
- Nostr controls/replies, when present: public event ids are kept as relay-verifiable anchors.
- Private operational evidence: represented as sanitized logical handles rather than raw logs.

## What feedback would help most

1. **Schema fit:** Are the five drift labels and seven event types compatible with the MemEvoBench annotation model, or should any be split/merged?
2. **Missing event types:** Do you need explicit `plan`, `decision`, `state_read`, `state_write`, or `verification` event types instead of deriving those from the current payload fields?
3. **Workflow mapping:** Which canonical MemEvoBench vocabulary should replace the provisional memory-system / agent-loop / trace-bench columns?
4. **Lineage fields:** Is the `{live_source, external_api, memory_state}` profile useful as-is, or should it be normalized into your own source taxonomy?
5. **Format adjustments:** If JSONL, per-event files, stricter JSON Schema, or a train/eval split would be easier, we can repackage the same sanitized events.

This v0.1 package is meant to be easy to criticize. The goal is not to claim these traces are universal; it is to make production memory-evolution failures concrete enough that MemEvoBench can decide what is useful, what is noisy, and what should be instrumented differently in the next iteration.
