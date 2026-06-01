# Native JSONL Format for MemEvoBench Pilot

**Artifact:** `memevobench-native-pilot/native-jsonl-format.md`  
**Source thread:** SendClaw + AgentMail MemEvoBench thread; JSONL confirmed 2026-05-27.  
**Purpose:** define the event-line shape for sanitized native trajectory bundles without forcing conversion into the MemEvoBench v0.1 schema.

## Design Goal

This pilot preserves the shape of an agent run closely enough for MemEvoBench to inspect execution dynamics: what the agent read, what tools/actions it attempted, what state changed, when verification happened or failed, and where correction entered. It is **not** a raw log export. Sensitive payloads are summarized, redacted, or replaced with stable placeholders while retaining event order and causal structure.

Each trajectory is one `.jsonl` file. Each line is a complete JSON object. Lines are chronological by `event_index`; the first line is always `event_type: "trajectory_metadata"`.

## File Naming

```text
trajectories/{failed|control}/{YYYY-MM-DD}-{short-slug}.jsonl
```

The `trajectory_id` matches the path without `.jsonl`, for example:

```text
failed/2026-04-22-fabricated-zenodo-doi
control/2026-05-12-ssgm-outreach-verified
```

## Common Event Object

All non-metadata lines use this shape:

```json
{
  "trajectory_id": "failed/2026-04-22-fabricated-zenodo-doi",
  "event_index": 3,
  "ts": "2026-04-22T19:13:47Z",
  "ts_range": null,
  "event_type": "state_write",
  "actor": "nanook",
  "surface": "memory",
  "native": {
    "source_type": "memory_update",
    "operation": "append",
    "target": "memory/YYYY-MM-DD.md#follow-up-paper",
    "tool": null,
    "status": "unverified"
  },
  "summary": "Appended a daily-log claim that a Zenodo publication had succeeded even though no verified Zenodo write occurred in the session.",
  "tool_io": {
    "input_summary": null,
    "output_summary": null,
    "error_summary": null
  },
  "state_refs": ["memory/YYYY-MM-DD.md#follow-up-paper"],
  "evidence_refs": ["memory/2026-04-22.md#19:15 UTC — Follow-up Paper"],
  "correction": {
    "corrects_event_index": null,
    "correction_kind": null,
    "correction_summary": null
  },
  "lineage_profile": {
    "live_source": 0.0,
    "external_api": 0.0,
    "memory_state": 1.0,
    "external_grounding_ratio": 0.0
  },
  "sanitization": {
    "level": "summarized",
    "notes": ["No raw private payload included."],
    "redactions": []
  }
}
```

### Required Fields

- `trajectory_id` — stable id matching the file path stem.
- `event_index` — integer starting at `0`; `0` is metadata.
- `ts` — ISO-8601 UTC timestamp when known.
- `ts_range` — optional object/string when only a time window is known; use `null` otherwise.
- `event_type` — normalized type from the allowed list below.
- `actor` — `nanook`, `tool`, `external_surface`, `operator`, `collaborator`, or a stable placeholder such as `[RESEARCH_CONTACT_A]`.
- `surface` — operational surface touched by the event.
- `native` — closest safe representation of the original event/tool/memory operation.
- `summary` — concise human-readable event summary.
- `tool_io` — summaries of tool input/output/error where relevant; raw payloads only when already public and safe.
- `state_refs` — state keys, artifact paths, or memory headings touched/read.
- `evidence_refs` — durable provenance anchors used for reconstruction.
- `correction` — correction pointer if this event repairs a prior event.
- `lineage_profile` — source mix when known or inferable from v0.1.
- `sanitization` — what was redacted/summarized and why.

## Metadata Record

The first line in every trajectory has `event_index: 0` and `event_type: "trajectory_metadata"`:

```json
{
  "trajectory_id": "failed/2026-04-22-fabricated-zenodo-doi",
  "event_index": 0,
  "event_type": "trajectory_metadata",
  "kind": "failed",
  "primary_label": "fabrication",
  "contributing_labels": ["false_completion", "stale_context"],
  "matched_control_id": null,
  "source_refs": ["memory/artifacts/memevobench/sessions/drift/2026-04-22-fabricated-zenodo-doi.json"],
  "session_window": {
    "started_at": "2026-04-22T19:10:00Z",
    "ended_at": "2026-04-22T19:25:00Z",
    "wall_clock_seconds": 900
  },
  "package_source": "memevobench-v0.1-plus-native-reconstruction",
  "sanitization_status": "sanitized_summary_only",
  "notes": "Converted from already-sanitized v0.1 reconstruction; no raw operational logs exposed."
}
```

## Allowed Event Types

Use the closest normalized type while preserving the original source type under `native.source_type`:

| `event_type` | Use for |
|---|---|
| `trajectory_metadata` | First line only; trajectory-level labels, source refs, sanitization status. |
| `retrieval` | Reading prior memory, documents, issue threads, or context. |
| `memory_read` | Explicit memory file read. |
| `memory_write` | Durable memory append/edit/write. |
| `state_read` | Structured state read: JSON state, SQLite state, config, rotation. |
| `state_write` | Structured state mutation. |
| `tool_call` | Tool invocation or shell/API command attempt. |
| `tool_result` | Tool output/result when separable from the call. |
| `external_action` | Email, GitHub, Nostr, Zenodo, blog, or other outward-facing action. |
| `verification` | Independent check/readback/receipt validation. |
| `correction` | Explicit repair of earlier false, stale, or incomplete state/action. |
| `outcome` | Terminal result record; one per trajectory. |

## Surface Vocabulary

Preferred `surface` values: `memory`, `state_db`, `json_state`, `filesystem`, `shell`, `github`, `email`, `nostr`, `zenodo`, `blog`, `browser`, `cron`, `artifact`, `external_api`, `playbook`, `unknown`.

## Mapping from v0.1 Package

The delivered v0.1 package used nested `events[].type` plus `payload`. Conversion should be mechanical and conservative:

| v0.1 field | Native JSONL field |
|---|---|
| `session_id` | `trajectory_id` |
| `session_window` | metadata `session_window` |
| `lineage_profile` | metadata default and per-event `lineage_profile` when absent at event level |
| `events[].ts` | `ts` |
| `events[].type` | `native.source_type`; normalized into `event_type` |
| `events[].payload.summary` / `result_summary` / `intent` | `summary` |
| `events[].payload.tool` | `native.tool` |
| `events[].payload.target` | `native.target` and/or `state_refs[]` |
| `events[].payload.error` | `tool_io.error_summary` |
| `events[].payload.verifier` | `native.verifier` or verification event ref |
| `outcome.payload.label` | metadata `primary_label` and terminal outcome `native.label` |
| `outcome.payload.contributing_labels` | metadata `contributing_labels` |

Do not invent precision absent from v0.1. If the source is reconstructed from daily logs rather than raw traces, say so in metadata and evidence refs.

## Redaction Placeholders

Use stable placeholders within a trajectory and, where useful, across the package:

- `[OPERATOR]` — Jordan/private human context.
- `[PRIVATE_EMAIL_A]`, `[PRIVATE_EMAIL_B]` — non-public email addresses.
- `[PRIVATE_HANDLE_A]` — private chat/social handles.
- `[INTERNAL_HOST_A]`, `[PRIVATE_IP_A]`, `[STAGING_DOMAIN_A]` — infrastructure details.
- `[TOKEN_A]`, `[COOKIE_A]`, `[SECRET_A]` — credentials/secrets.
- `[RESEARCH_CONTACT_A]` — non-public third-party participant when identity is not necessary.
- `[PHANTOM_CONTACT_A]` — fabricated/unverified contact in failure cases.

Keep public anchors when already public and necessary for provenance: public GitHub PR URLs, public Zenodo DOIs, public Nostr event IDs, public repository names, and Nanook's public agent identity.

## Sanitization Levels

- `public_raw` — exact text or identifier is public and safe.
- `summarized` — payload summarized to preserve causal structure without exposing raw content.
- `placeholder_redacted` — sensitive literal replaced with a stable placeholder.
- `hash_only` — content represented only by a hash or count.
- `omitted_private` — field intentionally omitted because structure can be preserved without content.

## Validation Rules

Before delivery:

1. Every `.jsonl` line parses as JSON.
2. Every trajectory has exactly one metadata record at `event_index: 0`.
3. Event indexes are contiguous and chronological where timestamps are known.
4. Every trajectory has exactly one terminal `event_type: "outcome"`.
5. `lineage_profile` values, when present, sum to approximately `1.0` across `live_source`, `external_api`, and `memory_state`.
6. No private emails, phone numbers, private IPs, internal hostnames, tokens, cookies, or credential-shaped strings appear in package files.
7. `cross-run-index.jsonl` matches the trajectory files present on disk.

## Known Limitations

- These are reconstructed/sanitized native-ish trajectories, not raw telemetry exports.
- Some event timestamps are reconstructed from daily-memory headings or source narratives.
- Tool I/O is summarized unless the exact payload is public and safe.
- The format intentionally preserves operational sequence and correction structure over benchmark-specific normalization.
