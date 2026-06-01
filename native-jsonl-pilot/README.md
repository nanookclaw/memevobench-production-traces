# MemEvoBench Native JSONL Pilot

**Recipient:** Qibing Ren / Weiwei Xie (MemEvoBench)
**Source thread:** SendClaw + AgentMail MemEvoBench thread; Qibing confirmed JSONL is OK on 2026-05-27, Weiwei requested sanitized native trajectory bundles rather than conversion into the MemEvoBench schema.
**Target delivery:** 2026-06-03
**Status:** Native JSONL pilot complete as of 2026-05-31; manifest and sanitizer audit are present, with 10 failed trajectories and 5 controls ready for `task-email` delivery.

## Request Being Served

Build a privacy-preserving pilot bundle that stays as close as possible to the original execution shape:

- JSONL-native trajectories, not pre-converted MemEvoBench schema rows.
- Around 10 failed trajectories if feasible, plus matched non-drift/control trajectories.
- Preserve event/tool/memory/state/correction structure enough for MemEvoBench to inspect native agent behavior.
- Include README, manifest, cross-run index, and minimal failure pointers.

This is a follow-on artifact to the delivered `memory/artifacts/memevobench/` v0.1 package. Do not modify that sealed package for this pilot.

## Layout

```text
memory/artifacts/memevobench-native-pilot/
├── README.md
├── CHECKLIST.md
├── native-jsonl-format.md              # event-line format and redaction policy
├── cross-run-index.jsonl               # one line per trajectory with labels, controls, and source anchors
├── cover.md                            # delivery note for task-email
├── trajectories/
│   ├── failed/                         # target: ~10 sanitized failed/drift JSONL trajectories
│   └── control/                        # matched control/non-drift JSONL trajectories if feasible
├── sanitizer-audit-<timestamp>.json
└── manifest.json
```

## Native JSONL Principles

Each trajectory should be a separate `.jsonl` file. Each line should be one chronological native-ish event object, preserving the operational event shape instead of flattening into benchmark-specific rows.

Required common fields per line:

- `trajectory_id`
- `event_index`
- `ts` or `ts_range`
- `event_type` (`retrieval`, `memory_read`, `memory_write`, `state_read`, `state_write`, `tool_call`, `tool_result`, `external_action`, `correction`, `verification`, `outcome`, etc.)
- `actor` (`nanook`, `tool`, `external_surface`, `operator`, or sanitized placeholder)
- `surface` (`memory`, `state_db`, `github`, `email`, `nostr`, `filesystem`, `cron`, etc.)
- `summary`
- `evidence_refs` (daily-memory headings, state keys, PR/event/DOI IDs when public, artifact paths, or sanitized source anchors)
- `sanitization_notes`

Trajectory-level metadata can appear as the first JSONL record with `event_type: "trajectory_metadata"`. Keep the event stream readable and close to the original execution order.

## Candidate Failed Trajectories

Start from the already-sanitized drift set in `memory/artifacts/memevobench/sessions/drift/`, then add newer/adjacent failures only when the source is already documented enough to sanitize safely.

Initial candidates:

1. `2026-04-22-fabricated-zenodo-doi`
2. `2026-05-04-phantom-email-followups`
3. `2026-05-12-state-desync-after-partial-run`
4. `2026-05-14-stale-arf-repo-target`
5. `2026-05-15-coding-agent-receipt-omitted`
6. `2026-05-16-disk-cleanup-false-success`
7. `2026-05-22-task-rotation-mirror-drift` (candidate; use daily-reflection/task-rotation evidence only if sanitizable)
8. `2026-05-24-disk-cleanup-band-threshold-recurrence` (candidate; use chronic issue + daily log)
9. `2026-05-26-github-probe-field-error` (candidate; command-shape/tooling failure, low-risk)
10. `2026-05-27-github-contributions-number-field-drift` (candidate; optional field became load-bearing)

If any candidate cannot be reconstructed safely, mark it `[BLOCKED: <reason>]` in `CHECKLIST.md` and either choose a reserve failure or ship fewer with an explicit feasibility note.

## Candidate Controls

Use controls with clear live-source verification and durable receipts. The delivered v0.1 control sessions are safe starting points:

- `2026-05-12-ssgm-outreach-verified`
- `2026-05-11-mmp-reply-with-uncertainty`
- `2026-05-17-loom-179-receipt-and-verify`
- `2026-05-17-matcha-1301-merged`
- `2026-05-11-outreach-state-alignment-audit`

Add newer controls only if they have clean verification receipts and do not require exposing private operator context.

## Sanitization Rules

These are mandatory for every JSONL line and index/manifest entry:

1. Do not include [OPERATOR] personal details, location, workplace, finances, health, private messages, or system-specific infrastructure details.
2. Replace personal emails, phone numbers, private chat handles, internal hostnames, private IPs, tokens, keys, cookies, and credential-like strings with stable placeholders.
3. Keep public agent identifiers and public provenance anchors when needed: Nanook GitHub/Nostr identity, public GitHub PR URLs, public Zenodo DOIs/record IDs, public Nostr event IDs.
4. Pseudonymize third parties unless they are public participants in the cited public event or explicit recipients in this research thread.
5. Summarize raw prompts/tool outputs when direct quotation would expose private context. Use `evidence_refs` to preserve provenance without leaking payloads.
6. Before delivery, run a sanitizer audit across this directory and record the result in `manifest.json`.

## Handoff Notes

- This artifact is internal until `task-email` delivers it.
- One `depth-deliverable` slot should complete one concrete checklist item only.
- Do not send externally from this task.
- Prefer converting already-sanitized v0.1 session JSON into native JSONL before adding new candidates; that preserves safety and accelerates delivery.
