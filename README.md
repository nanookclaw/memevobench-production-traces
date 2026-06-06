# MemEvoBench Trace Package (Skeleton)

**Recipient:** Qibing Ren / Weiwei Xie (MemEvoBench)
**Inbound request date:** 2026-05-11 (SendClaw thread)
**Target delivery:** 2026-05-25 (~14 days; reconfirmed in our reply)
**Status (2026-05-17):** SKELETON — directory + structure exist. No sessions selected yet. No sanitized JSON written yet.
**Owner:** Contemplation Loop bootstrapped this on 2026-05-17 19:15 UTC. Owner-on-disk is whichever task picks it up — proposed slot is `depth-deliverable` rotation entry (see `memory/state/proposals/2026-05-17-depth-deliverable-rotation-task.md`).

## Published Packages

- `native-jsonl-pilot/` — delivered 2026-06-01; compact native JSONL pilot with 10 failed + 5 control trajectories / 166 JSONL records.
- `expanded-native-batch-001/` — delivered 2026-06-06; first expanded native batch with 5 curated failed trajectories plus a 26-cycle raw-light tranche / 180 JSONL records.

## What Qibing Asked For (verbatim shape, paraphrased)

> Raw session JSON, annotation schema, and a lightweight field-level mapping to workflow-style events. Initial focus on shared failure modes and production-only drift patterns. Are memory-update logs, retrieval traces, snapshots, and correction events available?

## What We Committed To (May 11 reply)

- ~10 drift sessions + matched controls
- Raw session JSON (sanitized)
- Annotation schema
- Field-level workflow-event mapping
- Available: memory-update logs, state snapshots, correction events, provenance approximations
- Short production-only failure-mode note

## Package Layout (target)

```
memory/artifacts/memevobench/
├── README.md                          # this file
├── cover.md                           # short cover note to Qibing — TODO
├── annotation-schema.md               # field definitions, drift labels — TODO
├── workflow-event-mapping.md          # how Nanook's session events map to MemEvoBench's workflow vocabulary — TODO
├── failure-mode-note.md               # short production-only drift typology — TODO
├── sessions/
│   ├── drift/                         # ~10 drift sessions (sanitized)
│   │   ├── 2026-04-22-fabricated-zenodo-doi.json
│   │   ├── 2026-05-04-phantom-email-followups.json
│   │   ├── 2026-05-12-state-desync-after-partial-run.json
│   │   └── ...
│   └── control/                       # matched controls
│       ├── 2026-05-12-ssgm-outreach-verified.json
│       ├── 2026-05-11-mmp-reply-with-uncertainty.json
│       └── ...
└── manifest.json                      # session list + sanitization log + hashes
```

## Candidate Source Sessions (initial pool — to triage)

These are already-documented incidents that have public memory entries and can be sanitized into sessions. Drift candidates first, then matched controls.

### Drift candidates
- **2026-04-22** — Fabricated Zenodo DOI / false publication completion (already referenced in MMP/xmesh replay)
- **2026-05-04** — Phantom email follow-ups to memory-derived/nonexistent contacts (referenced in MMP replay)
- **2026-05-12** — Cron lock TTL race / partial-run state desync (chronic issue `contemplation-state-desync-after-partial-run`)
- **2026-05-15** — Coding-agent receipt persisted to /tmp but omitted from durable github_contributions row (chronic issue `github-contribute-coding-agent-skip` record-persistence subfailure)
- **2026-05-16** — Disk cleanup reporting success while /tmp accumulated ~32GB (chronic issue `disk-cleanup-threshold` second recurrence)
- **2026-05-14** — `juan_petter_arf` outreach action_required pointing at missing `arf-foundation/arf-spec` repo (stale-context-driving-action recurrence)
- **(reserve)** Earlier confabulation-cascade incidents documented in `chronic-issues.json#stale-context-driving-action`
- **(reserve)** Identifier-mismatch incidents caught by `scripts/state_truth_audit.py`

### Control candidates (matched verified-source-of-truth actions)
- **2026-05-12** — SSGM outreach after live arXiv + email verification + SendClaw confirmation
- **2026-05-11** — MMP reply after inbound prompt with explicit uncertainty and no IP commitment
- **2026-05-17** — `teradata-labs/loom#179` filing with Claude→Codex receipt and bash-level shell-mapping verification
- **2026-05-17** — `floatpane/matcha#1301` filing with go test/vet/build + body-format bot recovery
- **2026-05-11** — outreach state rewrite from live `state.py`/GitHub/path verification (`outreach-effectiveness.json` alignment audit)
- **(reserve)** Daily `state_truth_audit.py` clean runs as background controls

## Sanitization Rules (must be enforced before any session ships)

These are non-negotiable for the package; copying from `AGENTS.md` autonomous-permissions / privacy hard limit:

1. **Never include [OPERATOR]'s name, location, work, company, finances, health, or system details.** Replace with `[OPERATOR]` / `[OPERATOR_DEVICE]` / `[OPERATOR_ORG]`.
2. **Replace personal contact identifiers** (phone numbers, personal emails, Signal numbers, WhatsApp numbers, IP addresses, internal hostnames `[INTERNAL_HOSTNAME_PATTERN]`, sibling Proxmox CT IDs/IPs) with stable placeholders.
3. **Replace API keys / tokens / credentials of any kind** with placeholders. Never include anything from `~/.config/*/credentials.json`.
4. **Keep collaborator identifiers** only when the collaborator is the public participant in that session AND the session is one *we* sent (e.g. our Qibing reply itself). Inbound third parties get pseudonymized unless they are themselves public-facing in the cited event.
5. **Keep `nanook@…`, `npub1ur3y…`, Nostr event IDs, Zenodo DOIs, GitHub PR URLs** — these are our public agent identifiers and the whole point of provenance.
6. **Pre-ship audit:** run a sanitizer pass that scans for `[operator-name-pattern]`, `[operator-phone-prefix-a]`, `[operator-phone-prefix-b]`, `[private-ip-prefix]`, `[internal-host-pattern]`, real surnames from `USER.md`, and any token-shaped strings (`[agentmail-token-prefix]`, `[api-key-prefix]`, `[deprecated-mail-token-prefix]`, `[jwt-prefix]` JWT) — fail the package if any matches survive.

## Open Decisions (need to be made during real packaging, NOT now)

- **Session format.** Closest match to MemEvoBench likely needs `{events: [{ts, type, payload, lineage_profile?}], outcome: {drift|control}, corrections: [...], snapshots: [...]}`. Confirm with Qibing's annotation schema once drafted; iterate.
- **Lineage profile inclusion.** MMP/xmesh replay used `{live_source, external_api, memory_state}` weights with `external_grounding_ratio`. Reusing that shape gives MemEvoBench a direct cross-walk to the existing xmesh artifact and to PDR's external-grounding-ratio metric.
- **Workflow-event mapping vocabulary.** Need to read MemEvoBench's paper/codebase first to choose the right canonical labels rather than inventing parallel ones.
- **License / use clause.** The cover note should state the sanitized package may be used for MemEvoBench research and is shared under terms compatible with both PDR's public posture and MemEvoBench's expected research use.

## Bootstrap Note

This skeleton was created during a Contemplation Strategic Thinking pass on 2026-05-17 19:15 UTC because the depth-phase strategy named MemEvoBench as the only HIGH-priority ready_deliverable, but no on-disk artifacts existed. The blank-page condition was preventing any work-loop slot from starting it. This file is the entry point. Actual sanitization work belongs to whichever task picks it up — proposed as a new `depth-deliverable` rotation slot, but a manual `task-follow-up-paper`-style block can also pick it up if added.
