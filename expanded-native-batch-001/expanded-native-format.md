# Expanded Native JSONL Format Delta — Batch 001

**Artifact:** `memevobench-expanded-native/expanded-native-format.md`  
**Source thread:** AgentMail thread `0a3a7aad-7b4d-4edf-9659-f101567e32ed` requesting expedited expanded native/raw sanitized exports.  
**Purpose:** document the small format delta from the delivered native JSONL pilot for the first expanded MemEvoBench batch.

## Scope

This batch keeps the pilot format as the base contract. The change is not a schema rewrite; it is a tighter envelope for the fields Weiwei asked for explicitly:

1. safer model/component metadata,
2. clearer causal links across memory, state, tool, and correction events,
3. explicit post-memory-evolution drift annotations, and
4. a lightweight inventory/index that makes the batch easy to inspect before deeper benchmark processing.

The expanded export remains sanitized reconstructed/native JSONL, not raw private operational logs.

## Metadata Record Additions

Each trajectory still starts with `event_type: "trajectory_metadata"` at `event_index: 0`. Batch 001 metadata should add these fields when known and safe:

```json
{
  "trajectory_id": "failed/2026-06-XX-short-slug",
  "event_index": 0,
  "event_type": "trajectory_metadata",
  "kind": "failed",
  "primary_label": "stale_context",
  "secondary_labels": ["memory_evolution_drift", "false_completion"],
  "model": {
    "family": "codex-gpt55",
    "name": "gpt-5.5",
    "reasoning": "high_or_unknown",
    "known_from": "runtime_context|daily_log|unknown",
    "safety_note": "Only include if visible in non-private operational metadata."
  },
  "components": [
    {
      "name": "memory_search|daily_memory|json_state|github_workflow|cron|email|nostr|artifact_builder",
      "role": "seed|amplifier|missed_gate|correction_surface|verification_surface",
      "safe_detail": "short non-secret description"
    }
  ],
  "causal_chain_summary": "One sentence connecting seed state/event to later drift and correction.",
  "memory_evolution": {
    "involved": true,
    "mechanism": "stale_note_reused|missing_write|incorrect_state_persisted|mirror_field_drift|retrieval_gap",
    "first_bad_state_event_index": 2,
    "first_corrective_event_index": 8
  },
  "source_thread": "AgentMail thread 0a3a7aad-7b4d-4edf-9659-f101567e32ed",
  "sanitization_status": "sanitized_summary_only"
}
```

### Model Metadata Rules

- Include model family/name only when it is already present in safe runtime metadata, daily-log receipts, or public PR/CI notes.
- Use `unknown` rather than inferring from nearby sessions.
- Do not include provider API keys, account identifiers, internal routing details, hostnames, private IPs, or raw system prompts.
- If a trajectory spans multiple agents/tools, represent them as component metadata rather than pretending there was one model.

## Event-Level Causal Fields

Non-metadata events keep the pilot shape and may add these optional fields:

```json
{
  "causal_role": "seed|amplifier|missed_gate|correction|verification|outcome|context",
  "propagates_from": [1, 3],
  "state_boundary": "read|write|mirror_write|external_write|verification_read|none",
  "memory_evolution_note": "How this event changed, reused, or failed to update persistent memory/state.",
  "would_have_helped": "Specific signal that would have caught the drift earlier."
}
```

Use these fields sparingly. They are for reconstruction clarity, not for over-explaining every line.

## Post-Memory-Evolution Drift Annotation

For cases selected because a prior memory/state update later affected behavior, include:

- `memory_evolution.involved: true` in metadata.
- A `seed` event for the original persisted state, omitted write, or stale mirror.
- One or more `amplifier` events where the agent read/reused that state.
- A `missed_gate` event when verification should have happened but did not, if visible.
- A `correction` event pointing back with `correction.corrects_event_index` and/or `propagates_from`.
- A terminal `outcome` that distinguishes user-visible failure, internal caught-before-send failure, or near-miss.

If the causal chain is plausible but not proven, say so in `causal_chain_summary` and keep `memory_evolution.mechanism` conservative.

## Inventory Fields

`inventory/batch-001-inventory.md` should be human-readable and include one row per planned trajectory:

| Field | Meaning |
|---|---|
| `trajectory_id` | planned JSONL path stem |
| `kind` | failed/control/mixed |
| `primary_label` | main failure or control label |
| `memory_evolution` | yes/no + short mechanism |
| `components` | coarse surfaces involved |
| `model_metadata` | known/partial/unknown |
| `source_refs` | daily memory/chronic/artifact/public refs |
| `sanitization_status` | planned/in_progress/pass/blocked |
| `notes` | one sentence on why useful |

## Cross-Run Index Fields

`cross-run-index.jsonl` should contain one object per planned Batch 001 trajectory:

```json
{
  "trajectory_id": "failed/2026-06-XX-short-slug",
  "kind": "failed",
  "primary_label": "stale_context",
  "secondary_labels": ["memory_evolution_drift"],
  "causal_chain_summary": "Persistent stale state was read as current truth, then corrected after live verification.",
  "memory_evolution": {
    "involved": true,
    "mechanism": "stale_note_reused"
  },
  "components": ["memory", "json_state", "verification"],
  "model_metadata_status": "known|partial|unknown",
  "source_refs": ["memory/YYYY-MM-DD.md#heading", "memory/state/chronic-issues.json#id"],
  "planned_output": "trajectories/failed/2026-06-XX-short-slug.jsonl",
  "status": "planned|drafted|validated|blocked",
  "sanitization_status": "planned|pass|blocked"
}
```

The index should validate line-by-line as JSONL before delivery.

## Validation Additions

Batch 001 inherits the pilot validation rules and adds:

1. Metadata `components[]` must use coarse component categories, not private paths or host details.
2. `model.name` must be `unknown` unless supported by a safe source ref.
3. Any event with `propagates_from` must point to an earlier `event_index` in the same file.
4. If `memory_evolution.involved` is true, at least one event should have `causal_role: "seed"` and one later event should have `causal_role: "correction"`, `"verification"`, or `"outcome"`.
5. Inventory rows, cross-run-index records, and trajectory files must agree on `trajectory_id`, `kind`, and `primary_label`.
6. Sanitizer audit must cover this expanded directory only, not the sealed pilot directory.

## Known Limits

- Some trajectories will have partial model metadata because older daily logs did not always record a model.
- Causal annotations are reconstructed from durable receipts, not raw execution traces.
- The goal is fast, useful incremental export. Prefer conservative labels and clear provenance over complete telemetry.
