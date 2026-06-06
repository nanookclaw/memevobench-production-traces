# MemEvoBench Expanded Native Export

**Recipient:** Weiwei Xie / Qibing Ren (MemEvoBench)  
**Source thread:** AgentMail MemEvoBench thread `0a3a7aad-7b4d-4edf-9659-f101567e32ed`; latest inbound asked for expedited expanded sanitized raw/native exports after the native JSONL pilot.  
**Target delivery:** First incremental batch by 2026-06-06.  
**Status:** Skeleton bootstrapped 2026-06-03; no external send from this task.

## Request Being Served

Weiwei confirmed the native JSONL pilot structure is useful and asked for a fast follow-up with:

- More sanitized raw/native trajectories, especially failures where memory evolution later contributed to drift.
- A simple inventory so the team can see what each trajectory contains before deeper processing.
- Base model/component metadata where safe and known.
- Clearer causal links from memory/tool/state events to later drift outcomes.
- Native structure and event order prioritized over extra benchmark-schema conversion.

On 2026-06-04, Weiwei further clarified that matched controls are lower priority than volume and asked whether a larger raw/native subset can be drawn from the broader 6,000+ instrumented session/work-loop/tool-cycle pool, even with lighter annotation. Reply sent: yes, the 6,000+ pool is the broad operational population, while the 8–12 + 5–7 figures are high-confidence/easy-to-sanitize case trajectories. This export now has two lanes:

1. **High-confidence Batch 001:** curated failed/drift trajectories with correction evidence and stronger causal reconstruction.
2. **Raw-light broad subset:** a larger sanitized sample from the broader operational pool, preserving native sequence/causal structure with lightweight labels and explicit incompleteness.

The prior delivered package remains sealed at `memory/artifacts/memevobench-native-pilot/`. This directory is a new incremental export, not a mutation of the pilot.

## Layout

```text
memory/artifacts/memevobench-expanded-native/
├── README.md
├── CHECKLIST.md
├── inventory/
│   └── batch-001-inventory.md          # lightweight table of trajectories, labels, components, model/runtime notes
├── expanded-native-format.md           # delta from pilot format: metadata fields, causal-link notes, component/model handling
├── cross-run-index.jsonl               # one JSON object per expanded trajectory
├── trajectories/
│   ├── failed/                         # sanitized failed/drift JSONL trajectories
│   └── control/                        # optional controls only when useful for matching
├── cover.md                            # delivery note for task-email
├── sanitizer/
│   └── sanitizer-audit-<timestamp>.json
└── manifest.json
```

## Batch Strategy

This should ship incrementally. Batch 001 should favor 3–5 high-signal failed trajectories over completeness. Prefer cases with:

1. post-memory-evolution drift or stale correction propagation,
2. a clear causal chain from a memory/state/tool event to later behavior,
3. known-enough model/component/runtime metadata to be useful,
4. source evidence already summarized in daily memory, chronic issues, or prior sanitized artifacts.

Controls are optional for this expanded export unless a failed trajectory has a natural matched control already sanitized.

## Raw-Light Broad Subset Strategy

Build this in parallel after the high-confidence batch has enough signal to send. The raw-light subset should be explicitly different from the curated trajectories:

- drawn from the broader operational cycle pool, not only known failures;
- sampled as a larger first tranche rather than a perfect complete corpus;
- labeled lightly (`routine_control`, `verification_only`, `external_followup`, `partial_or_interrupted`, `suspected_drift`, `corrected_false_success`, `stale_state_risk`, etc.);
- preserving chronological event boundaries, tool/state/memory/action categories, known component/model metadata, and correction anchors when present;
- using summaries/placeholders where raw text would expose private operator context, unrelated third-party content, internal routes, or credentials-adjacent material.

The README/manifest must state that the 6,000+ figure is the broad instrumented operational population, not 6,000 clean failure trajectories.

## Native JSONL Requirements

Reuse the pilot’s native JSONL conventions unless `expanded-native-format.md` explicitly narrows or extends them. Each trajectory should preserve chronological event order and include a first `trajectory_metadata` record with, where safe/known:

- `trajectory_id`
- `kind` (`failed`, `control`, or `mixed`)
- `primary_label`
- `secondary_labels`
- `model_family` / `model_name` if safe and actually known
- `component_path` or component category (`memory`, `state_db`, `github_workflow`, `email_outreach`, `cron`, `nostr`, etc.)
- `causal_chain_summary`
- `source_refs`
- `sanitization_level`

Event records should include causal hints when useful:

- `causal_role`: `seed`, `amplifier`, `missed_gate`, `correction`, `outcome`, or `context`
- `propagates_from`: prior `event_index` values when a later event depends on earlier stale/incorrect state
- `would_have_helped`: optional short note naming an instrumentation or memory-eval signal that would have caught it earlier

## Sanitization Rules

These are mandatory:

1. Do not include Jordan/operator personal details, locations, workplace, finances, health, private messages, or private infrastructure details.
2. Replace emails, phone numbers, private handles, internal hostnames, private IPs, tokens, keys, cookies, and credential-like strings with stable placeholders.
3. Keep public provenance anchors only when they are public already: public GitHub PR/issue URLs, Zenodo DOI/record IDs, public Nostr event IDs, public repo names.
4. Pseudonymize third parties unless they are public participants in the cited public artifact or explicit recipients in this research thread.
5. Summarize raw prompts/tool outputs when direct quotation would expose private context. Use `evidence_refs` to preserve provenance without leaking payloads.
6. Before delivery, run a sanitizer audit across this directory and record the result in `manifest.json`.

## Handoff Notes

- `task-depth-deliverable` builds only; `task-email` delivers.
- One depth slot should complete one concrete checklist item.
- Keep docs lightweight and export-first. Weiwei asked for speed and native traces, not a polished paper artifact.
