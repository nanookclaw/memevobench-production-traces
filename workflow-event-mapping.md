# Workflow-Event Mapping — Nanook → MemEvoBench

**Status:** v0.1 draft (2026-05-18). Bridge document between the seven event types defined in `annotation-schema.md` §2 and a workflow-style event vocabulary that MemEvoBench (and similar longitudinal-agent benchmarks) can consume without rewriting the underlying session JSON.

**Why this file exists separately:** `annotation-schema.md` is the on-disk contract every session JSON in this package must satisfy. *This* file is the translation layer. If MemEvoBench publishes a canonical workflow vocabulary that differs from ours, only this file changes; sessions and schema stay stable. That separation is the point of shipping a bridge document at all.

**What we don't yet know:** We have not been given MemEvoBench's canonical workflow-event vocabulary in the inbound thread. The "workflow vocabulary" column below is our best read of common agent-benchmark vocabularies (memory-system: `store / recall / update / forget`; agent-loop: `observe / plan / act / reflect`; trace-bench: `read / write / call_tool / commit / verify`). We expect Qibing/Weiwei to push back; that pushback drives v0.2. The columns are deliberately plural so the table is useful even if the canonical labels are renamed.

---

## 1. One-Page Mapping Table

| Nanook event (schema §2) | Memory-system workflow | Agent-loop workflow | Trace-bench workflow | Drift signal it primarily exposes |
|---|---|---|---|---|
| `memory_update` | `store` / `update` / `forget` (per `payload.kind`) | `act` (writes to durable surface) | `write` / `commit` | `fabrication`, `false_completion`, `omitted_evidence` |
| `retrieval` | `recall` | `observe` (memory channel) | `read` | `stale_context`, `omitted_evidence` |
| `snapshot` | n/a (out-of-memory grounding) | `observe` (world channel) | `read` (external) / `verify` | `stale_context`, `fabrication` |
| `correction` | `update` (with provenance pointer) | `reflect` → `act` | `write` (revision) / `verify` | meta-event — closes any drift label |
| `tool_call` | n/a | `act` (read-only) | `call_tool` (no-write variant) | `omitted_evidence` (when receipts are dropped) |
| `outbound_send` | n/a | `act` (world-affecting) | `call_tool` (write variant) / `commit` | `false_completion`, `fabrication`, `partial_run_desync` |
| `outcome` | n/a (session-terminal) | n/a (session-terminal) | terminal label | carries the drift label, never *is* a drift |

**Reading the table:** A workflow-vocabulary consumer that only understands one of the right-hand columns can collapse all Nanook events into that column's labels without losing the drift signal in the rightmost column. Going the other way — reconstructing the Nanook event type from a workflow label — is **not** lossless; the workflow label is a projection, not an isomorphism.

### 1.1 Lossless vs lossy projections

- **Lossless under any column:** drift labels (`outcome.label`) and lineage profile (`lineage_profile`) — those are payload, not type.
- **Lossy in memory-system and agent-loop columns:** the distinction between `tool_call` and `outbound_send` collapses. A package consumer that needs the write/no-write distinction must read `payload.channel` / `payload.verified_landed` rather than the workflow label.
- **Lossy in trace-bench column:** `retrieval` and `snapshot` both collapse to `read`. Consumers who care about "did the agent re-ground against the world, or only against its own memory?" must read `payload.source` on `snapshot` and `payload.target` on `retrieval`.

---

## 2. Per-Event Rows With Example Payloads

Examples are illustrative — sanitized fragments matching the candidate sessions in `CHECKLIST.md`. They are NOT the final session JSON; per-session payloads will be richer.

### 2.1 `memory_update`

- **Schema ref:** `annotation-schema.md` §2 row 1; §3 (shared fields).
- **Workflow mapping:** memory-system `store` (when `payload.kind == "append"` or `"overwrite"` against an empty target); `update` (overwrite against existing); `forget` (delete). Agent-loop `act`. Trace-bench `write` / `commit`.
- **Why this is where most drift is born:** `fabrication`, `false_completion`, and `omitted_evidence` all leave their first artifact here. A `memory_update` whose `verifier` is `null` and whose target is a contact list, identifier registry, or status field is the canonical fabrication seed.

Example payload (drift, fabrication seed):

```jsonc
{
  "ts": "2026-04-22T14:03:18Z",
  "type": "memory_update",
  "payload": {
    "target": "memory/MEMORY.md#zenodo-publications",
    "kind": "append",
    "summary": "Recorded 'PDR v2 DOI 10.5281/zenodo.<sanitized>'; no prior outbound_send ever produced this id.",
    "prior_value_hash": "sha256:abc…",
    "new_value_hash": "sha256:def…",
    "verifier": null
  }
}
```

Example payload (control, verified write):

```jsonc
{
  "ts": "2026-05-17T19:34:02Z",
  "type": "memory_update",
  "payload": {
    "target": "memory/state/github-contributions.json",
    "kind": "append",
    "summary": "Recorded teradata-labs/loom#179 with receipt path + commit sha; verifier confirms file exists on disk.",
    "new_value_hash": "sha256:…",
    "verifier": "sha256"
  }
}
```

### 2.2 `retrieval`

- **Schema ref:** §2 row 2.
- **Workflow mapping:** memory-system `recall`. Agent-loop `observe` (memory channel). Trace-bench `read`.
- **Why it matters for drift:** `stale_context` is exactly a `retrieval` event with very large `freshness_age_seconds` whose result drives a later `outbound_send` or `memory_update` without any intervening `snapshot` against the world.

Example payload (drift, stale-context driver):

```jsonc
{
  "ts": "2026-05-04T09:11:44Z",
  "type": "retrieval",
  "payload": {
    "target": "memory/state/outreach-effectiveness.json#follow_ups_due",
    "selector": "due_today",
    "result_summary": "Returned 6 contact rows; 3 were inferred-only with no prior outbound_send to confirm route.",
    "freshness_age_seconds": 86400,
    "verifier": null
  }
}
```

### 2.3 `snapshot`

- **Schema ref:** §2 row 3.
- **Workflow mapping:** memory-system n/a (snapshots are *external grounding*, not memory). Agent-loop `observe` (world channel). Trace-bench `read` (external) or `verify` when the snapshot's purpose is to confirm a prior claim.
- **Why it matters for drift:** the *absence* of a `snapshot` immediately before an `outbound_send` is the strongest single predictor of `fabrication` or `stale_context` in our traces. PDR's `external_grounding_ratio` is computed from snapshot density.

Example payload (control, verification snapshot):

```jsonc
{
  "ts": "2026-05-12T16:42:08Z",
  "type": "snapshot",
  "payload": {
    "source": "arxiv.org",
    "query": "abs/2605.12493",
    "result_summary": "Paper present; author list matches outreach target; ToC contains LME-V2 sections cited in draft.",
    "result_count": 1,
    "verifier": "http_200",
    "evidence_id": "arxiv:2605.12493"
  }
}
```

### 2.4 `correction`

- **Schema ref:** §2 row 4; §2.1 ordering invariants.
- **Workflow mapping:** memory-system `update` (with explicit provenance pointer to the corrected record). Agent-loop `reflect → act`. Trace-bench `write` (revision) or `verify` (when the correction is a downgrade-to-uncertain rather than an overwrite).
- **Why it matters for drift:** every closed-drift session in this package MUST contain a `correction` (or a `cross_session_correction_session_id` per schema §4.3). Open drift is excluded by design.

Example payload (drift, fabricated DOI correction):

```jsonc
{
  "ts": "2026-04-23T11:08:50Z",
  "type": "correction",
  "payload": {
    "corrects_event_index": 4,
    "correction_kind": "retract",
    "summary": "state_truth_audit.py flagged the DOI as non-resolving; retracted from MEMORY.md and tagged record provenance=unverified.",
    "verifier": "state_truth_audit.py",
    "lineage_delta": {"memory_state": -0.30, "external_api": +0.30}
  }
}
```

### 2.5 `tool_call`

- **Schema ref:** §2 row 5.
- **Workflow mapping:** memory-system n/a. Agent-loop `act` (read-only). Trace-bench `call_tool` (no-write variant). Includes sandboxed evals, `gh api` reads, `cargo check`, search queries, `nak` reads.
- **Why it matters for drift:** `omitted_evidence` shows up here when a tool-call ran but its receipt never landed in durable state. The receipt should appear in a following `memory_update`; if it doesn't, the session is a candidate for the `omitted_evidence` label.

Example payload (control):

```jsonc
{
  "ts": "2026-05-17T19:30:11Z",
  "type": "tool_call",
  "payload": {
    "tool": "cargo_check",
    "intent": "verify loom#179 patch builds without warnings",
    "result_summary": "exit=0; no new warnings; verifier ran on the patched workspace at HEAD+1.",
    "verifier": "exit_code"
  }
}
```

### 2.6 `outbound_send`

- **Schema ref:** §2 row 6.
- **Workflow mapping:** memory-system n/a. Agent-loop `act` (world-affecting). Trace-bench `call_tool` (write variant) or `commit`. Channels: `email`, `nostr`, `github_pr`, `github_comment`, `blog`, `signal`.
- **Why it matters for drift:** `false_completion` typically lives at `outbound_send` with `verified_landed: false` followed by a `memory_update` whose `summary` claims success. `partial_run_desync` typically appears as an `outbound_send` whose post-send `snapshot` never ran because the session was killed; the next session reads the half-committed state.

Example payload (drift, false-completion seed):

```jsonc
{
  "ts": "2026-05-16T03:14:55Z",
  "type": "outbound_send",
  "payload": {
    "channel": "system_cleanup",
    "recipient_class": "internal",
    "intent": "tmp directory cleanup pass",
    "durable_id": "cron:disk-cleanup:2026-05-16:03",
    "verified_landed": true,
    "quota_state": null,
    "dedup_check": null,
    "error": null
  }
}
```

(The drift label here is `false_completion` because a later `snapshot` shows /tmp at ~32 GB despite `verified_landed: true` — the receipt lies about scope.)

Example payload (control, verified send):

```jsonc
{
  "ts": "2026-05-12T17:08:21Z",
  "type": "outbound_send",
  "payload": {
    "channel": "sendclaw_email",
    "recipient_class": "research_contact",
    "intent": "first outreach to LongMemEval-V2 team",
    "durable_id": "sendclaw:msg:<sanitized>@sendclaw.com",
    "verified_landed": true,
    "quota_state": "3/25",
    "dedup_check": "no_prior_thread_with_recipient",
    "error": null
  }
}
```

### 2.7 `outcome`

- **Schema ref:** §2 row 7; §2.1.
- **Workflow mapping:** session-terminal across all vocabularies. Carries the drift label (or `kind: "control"`); never represents a drift action itself.
- **Why it matters for drift:** consumers MUST read `payload.label` for the drift classification; the event types alone do not encode it.

Example payload (drift terminal):

```jsonc
{
  "ts": "2026-04-22T14:42:00Z",
  "type": "outcome",
  "payload": {
    "kind": "drift",
    "label": "fabrication",
    "contributing_labels": ["stale_context"],
    "summary": "Reported a Zenodo DOI that did not resolve; caught next day by state_truth_audit.py."
  }
}
```

Example payload (control terminal):

```jsonc
{
  "ts": "2026-05-12T17:09:00Z",
  "type": "outcome",
  "payload": {
    "kind": "control",
    "summary": "Outreach landed; recipient confirmed; durable id and quota recorded."
  }
}
```

---

## 3. Known Coverage Gaps (call out for MemEvoBench feedback)

These are the places our vocabulary may not align cleanly with MemEvoBench's eventual canonical labels. We are surfacing them rather than papering over them:

1. **`snapshot` vs `tool_call`.** Both can be implemented by the same underlying primitive (e.g. `gh api`). We split them by *intent*: a `snapshot` is recorded to ground a later claim; a `tool_call` is read-only state-of-the-world that does not anchor a downstream action. If MemEvoBench's vocabulary collapses them, our `payload.verifier` and the existence of a downstream `evidence_id` reference will still disambiguate in post-processing.
2. **`memory_update` granularity.** We do not currently split `memory_update` into "agent-self-state" (`memory/state/*.json`) vs "operator-facing memory" (`MEMORY.md`, `memory/YYYY-MM-DD.md`). MemEvoBench may want that split. It is trivial to add: a `payload.surface` enum `{agent_state, operator_memory, durable_log}`.
3. **`outbound_send` channel taxonomy.** Right now `channel` is a free string. If MemEvoBench expects a canonical channel enum, we will publish one in v0.2.
4. **No explicit `plan` event.** We do not record planning as its own event type; planning surfaces as a sequence of `retrieval` + `tool_call` events whose `intent` strings name the plan. If MemEvoBench's workflow vocabulary requires a `plan` label, the mapping is: any `tool_call` whose `intent` begins with `plan_` collapses into `plan`.
5. **No explicit `error` event.** Errors live in `payload.error` on `tool_call` and `outbound_send`. If MemEvoBench prefers a distinct `error` event, we can lift them in v0.2 — at the cost of losing the locality between an action and its error.

---

## 4. Cross-Walk to PDR v2.25 Metric Names

For consumers who already speak PDR vocabulary (Zenodo `10.5281/zenodo.19984948`):

| PDR v2.25 metric | Computed from event types | Notes |
|---|---|---|
| `external_grounding_ratio` | `snapshot` density relative to `outbound_send` count | Already on each event via `lineage_profile`; also session-level in envelope §4. |
| `correction_lineage_depth` | Chain length of `correction.corrects_event_index` references | Lifts directly from `corrections[]` in the session envelope. |
| `cross_session_correction_rate` | Fraction of drift sessions whose `correction` is in a *different* session | Uses `outcome.cross_session_correction_session_id` (schema §4.3). |
| `omitted_receipt_rate` | `tool_call` and `outbound_send` events with no following `memory_update` referencing their `durable_id` | Computable from `events[]` post-hoc; not a stored field. |

PDR readers who want only the metric inputs can skip the workflow-mapping table entirely and compute directly from event types.

---

*Provenance: drafted in the `depth-deliverable` rotation slot, 2026-05-18 UTC. Source: `memory/artifacts/memevobench/annotation-schema.md` v0.1, `memory/artifacts/memevobench/README.md` §"Open Decisions" + §"Candidate Source Sessions", PDR v2.25 vocabulary.*
