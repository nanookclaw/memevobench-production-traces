# Raw-Light Sampling Plan — MemEvoBench Expanded Native Export

**Artifact:** `memevobench-expanded-native/inventory/raw-light-sampling-plan.md`  
**Source thread:** AgentMail thread `0a3a7aad-7b4d-4edf-9659-f101567e32ed`  
**Created:** 2026-06-05  
**Purpose:** define the first broader sanitized/native tranche requested after the pilot, distinct from the curated Batch 001 failure trajectories.

## Request Served

Weiwei asked whether the larger raw/native subset can draw from the broader 6,000+ instrumented operational-cycle pool, even if the annotation is lighter than the 8–12 high-confidence and 5–7 matched-control cases. This plan makes that lane concrete before exporting `raw-light/index.jsonl` and a first tranche file.

The raw-light lane is **not** a claim that all 6,000+ cycles are clean benchmark-ready failures. It is a first sanitized sample from a broad operational population: work-loop, verification, state/memory maintenance, GitHub contribution, content, outreach, and interrupted/partial cycles. The goal is native sequence shape and provenance-preserving lightweight labels, not polished causal reconstruction for every row.

## First Tranche Scope

**Planned tranche id:** `raw-light/first-tranche-2026-06-05.jsonl`  
**Target size:** 20–40 cycle summaries for the first sendable tranche, expandable after feedback.  
**Source window:** 2026-05-24 through 2026-06-05, biased toward cycles already summarized in daily memory and state artifacts so sanitization is fast and reliable.  
**Granularity:** one record per operational cycle or compact multi-event episode, not full raw transcript dumps.

### Include

Prioritize records from these surfaces:

1. **Verification/audit cycles** — DOI/Nostr/state truth checks, false-dead prevention, sanitizer validations, manifest/state validation.
2. **Work-loop cycles** — skips, fall-through, blocked conditions, successful depth-deliverable steps, and provider/no-response recovery cases.
3. **Memory/state maintenance** — daily reflection insights, checklist/artifact advancement, state mirror repairs, root-cruft/stub handling.
4. **External follow-through summaries** — public GitHub PR lifecycle updates, CI/review follow-up, CLA/status handling, without private account/security details.
5. **Content/research cycles** — blog/MCP publish decisions, Nostr post/timeline decisions, outreach wait/defer decisions, when they expose useful state/action boundaries.

### Exclude from the first tranche

- Raw private prompts, full user messages, private contact details, credentials, tokens, cookies, or account recovery/security material.
- Internal hostnames, private IPs, staging URLs, SSH details, local service ports, or deployment commands that would reveal private infrastructure.
- Full email bodies unless the text is already intentionally prepared for external delivery and still safe after contact pseudonymization.
- Long GitHub diffs or raw CI logs; use public PR/check URLs or summarized status instead.
- Duplicate routine loops that add no new label coverage once a representative example is included.

## Lightweight Labels

Each raw-light record should use one `primary_label` and optional `secondary_labels` from this controlled set. Add new labels only if a record genuinely does not fit.

| Label | Use for |
|---|---|
| `routine_control` | Normal successful cycle with no drift or correction, useful as baseline structure. |
| `verification_only` | Cycle primarily reading/validating state or external identifiers without durable public action. |
| `external_followup` | Public artifact/PR/issue/status follow-through where the key behavior is maintaining continuity. |
| `depth_deliverable_progress` | Internal artifact-building step for an existing collaborator. |
| `blocked_skip_self_healing` | Single-sample external/tool failure that causes a skip only, not a durable rewrite. |
| `partial_or_interrupted` | Provider/tool/session failure where side effects may need reconstruction. |
| `suspected_drift` | Possible memory/state/tool drift where evidence is not strong enough for a curated failed trajectory. |
| `corrected_false_success` | Claimed success or completion was later corrected by verification. |
| `stale_state_risk` | Durable state/memory/file artifact could mislead later sessions if not checked. |
| `sanitizer_validation` | Record is mainly about proving sanitized output is safe/consistent. |
| `outreach_wait_state` | Correct action is waiting/defer because depth queue or counterpart response gates action. |
| `content_cooldown_decision` | Publishing/distribution decision controlled by cooldown, saturation, or target availability. |

## Record Shape

`raw-light/index.jsonl` should contain one compact object per selected cycle:

```json
{
  "raw_light_id": "rl-2026-06-05-001",
  "planned_output": "raw-light/first-tranche-2026-06-05.jsonl",
  "source_window": {"start": "2026-05-24", "end": "2026-06-05"},
  "source_refs": ["memory/2026-06-05.md#depth-deliverable"],
  "primary_label": "depth_deliverable_progress",
  "secondary_labels": ["sanitizer_validation"],
  "component_categories": ["daily_memory", "artifact_builder", "json_state"],
  "model_metadata_status": "partial",
  "privacy_level": "summary_only",
  "annotation_level": "raw_light",
  "status": "planned",
  "omissions": ["raw prompt", "private operator context"],
  "notes": "One sentence explaining why this cycle was selected."
}
```

The first tranche JSONL should preserve native order within each compact episode:

```json
{
  "raw_light_id": "rl-2026-06-05-001",
  "event_index": 0,
  "event_type": "cycle_metadata",
  "primary_label": "depth_deliverable_progress",
  "source_refs": ["memory/2026-06-05.md#depth-deliverable"],
  "sanitization_status": "sanitized_summary_only"
}
```

Then add 2–5 event records per cycle using event types such as `state_read`, `blocked_condition_check`, `tool_result_summary`, `state_write`, `artifact_write`, `verification`, and `outcome`. Use summaries instead of raw payloads when the original text contains private context.

## Sampling Method

1. Build a candidate table from daily memory headings and durable state records in the source window.
2. Deduplicate by underlying episode, not by repeated cron attempts. Example: multiple blocked Reddit 403 skips can collapse to one `blocked_skip_self_healing` example unless a later run changed behavior.
3. Balance the first tranche roughly as:
   - 25% routine/control or verification-only cycles,
   - 25% depth/artifact/state-maintenance cycles,
   - 25% external follow-through cycles,
   - 25% blocked/partial/suspected-drift cycles.
4. Prefer episodes with clear start/end timestamps and existing daily-log receipts.
5. Keep model metadata conservative: include model family/name only when already visible in safe runtime/daily-log metadata; otherwise use `unknown` or `not_material`.

## Sanitization Limits

Raw-light records are intentionally lighter than Batch 001:

- Causal fields may be absent or approximate unless the daily receipt already supports them.
- `source_refs` can point to daily memory headings or local artifact paths, but raw private content stays summarized.
- Public GitHub/Nostr/DOI references are allowed when already public and useful for provenance.
- Any record needing private credentials, private infrastructure context, or operator personal details to make sense should be marked `blocked` in `raw-light/index.jsonl` rather than reconstructed.

## Validation Gate Before Delivery

Before this lane is shipped:

1. `raw-light/index.jsonl` parses line-by-line as JSON.
2. Every `raw_light_id` in the tranche exists in the index and has matching labels.
3. Event indexes are contiguous within each `raw_light_id`.
4. Sensitive-pattern scan over `raw-light/`, `inventory/`, `README.md`, `CHECKLIST.md`, `cover.md`, and `manifest.json` returns zero private IPs/internal hosts/emails/phone numbers/token-like strings, except public-safe DOI/GitHub/Nostr identifiers.
5. `manifest.json` explicitly states that the raw-light lane is a sampled operational subset, not a complete 6,000-record export and not a set of 6,000 confirmed failures.

## Next Step

Create `raw-light/index.jsonl` with the first 20–40 selected cycle records and mark each as `planned` or `blocked` before drafting `raw-light/first-tranche-2026-06-05.jsonl`.
