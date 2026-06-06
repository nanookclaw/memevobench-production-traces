# Batch 001 Inventory — MemEvoBench Expanded Native Export

**Artifact:** `memevobench-expanded-native/inventory/batch-001-inventory.md`  
**Source thread:** AgentMail thread `0a3a7aad-7b4d-4edf-9659-f101567e32ed`  
**Purpose:** lightweight inspection table for planned Batch 001 expanded native trajectories before JSONL reconstruction.  
**Status:** candidate inventory drafted 2026-06-04; no external send from this task.

## Selection Notes

Batch 001 favors recently observed failures where persistent memory, durable state, or state-audit logic changed what later runs believed. These are reconstructed from daily memory, artifact receipts, and public-safe operational summaries. Raw private prompts, private infrastructure details, credentials, and operator-identifying data are intentionally excluded.

| trajectory_id | kind | primary_label | memory_evolution | components | model_metadata | source_refs | sanitization_status | notes |
|---|---|---|---|---|---|---|---|---|
| `failed/2026-06-01-zenodo-timeout-false-dead` | failed | `transient_external_read_misclassified_as_dead` | yes — verifier state treated slow/timeout responses as terminal death until corrected to NET/transient classification | `verification_script`, `zenodo_identifier_state`, `daily_memory`, `chronic_issue` | partial — run model not material; component metadata known from daily log | `memory/2026-06-01.md#L67-L83`; `scripts/state_truth_audit.py`; `memory/state/chronic-issues.json#audit-transient-network-misclassified-as-dead` | pass — JSONL written/validated 2026-06-04 | Good high-signal case for external identifier drift: one slow API window could have propagated false DEAD state into paper/distribution records without a confirmation gate. |
| `failed/2026-06-02-nostr-empty-relay-false-dead` | failed | `single_sample_notfound_false_dead` | yes — prior verifier fix covered timeout/error but not clean empty relay stdout; state would have evolved by deleting/rewriting live Nostr IDs | `verification_script`, `nostr_relay_query`, `daily_memory`, `chronic_issue` | partial — run model not material; component metadata known from daily log | `memory/2026-06-02.md#L68-L73`; `scripts/state_truth_audit.py`; `memory/state/chronic-issues.json#audit-nostr-single-window-notfound-misclassified-as-dead` | pass — JSONL written/validated 2026-06-04 | Complements the Zenodo case with a different external system and the same root pattern: terminal state claims from one insufficient sample. |
| `failed/2026-05-29-shell-markdown-mutation` | failed | `shell_expansion_corrupted_durable_memory_write` | yes — unquoted heredoc shell expansion mutated daily-log markdown before it became persistent memory; later fixed with write/edit-tool discipline | `daily_memory`, `shell_write_path`, `github_workflow`, `playbook_rule` | partial — daily log records workflow; model not necessary for reconstruction | `memory/2026-05-29.md#L63-L65`; `memory/2026-05-29.md#L158-L173`; `memory/state/chronic-issues.json#shell-markdown-mutation` | pass — JSONL written/validated 2026-06-04 | Useful because the failure is not external API behavior: durable memory itself was corrupted by the write mechanism, then corrected before user-visible drift compounded. |
| `failed/2026-06-03-workloop-provider-no-response` | failed | `silent_partial_run_without_durable_receipt` | yes — large work-loop completions failed before durable memory entries, creating a gap where possible tool actions had to be reconstructed by spot-check rather than normal receipts | `cron_work_loop`, `daily_memory`, `task_rotation`, `chronic_issue`, `artifact_state` | known/partial — provider family recorded in cron-health daily log; exact child execution internals not needed | `memory/2026-06-03.md#L176-L192`; `memory/state/chronic-issues.json#workloop-transient-provider-no-response-consecutive-errors` | pass — JSONL written/validated 2026-06-04 | Captures receipt absence as a memory-evolution failure: later runs needed explicit duplicate/partial artifact checks because the normal write-down step never happened. |
| `failed/2026-06-03-empty-state-db-stubs` | failed | `derived_state_stub_created_by_wrong_cwd` | yes — empty SQLite files in plausible state paths could mislead future local probes if treated as canonical; corrected by archiving stubs and re-verifying canonical DB path/count | `json_state`, `sqlite_state`, `workspace_hygiene`, `daily_memory`, `chronic_issue` | partial — component metadata safe; model not material | `memory/2026-06-03.md#L102-L119`; `memory/state/chronic-issues.json#workspace-derived-state-and-root-cruft-drift`; `scripts/state.py` | pass — JSONL written/validated 2026-06-05 | High value for MemEvoBench because the drift is embodied in filesystem state: zero-byte lookalike DBs at wrong paths created a future false-source-of-truth hazard. |

## Planned Batch 001 Shape

- Start with the first three trajectories above if time is tight; they are the clearest post-memory-evolution drift cases.
- Add the work-loop no-response and empty-state-stub cases if sanitizer and reconstruction checks pass before the delivery window.
- Controls are optional for this expanded batch; prefer five strong failed trajectories over weak matched controls unless an existing sanitized control maps naturally to verifier/checkpoint behavior.

## Sanitization Plan

- Replace any private hostnames, private IPs, account identifiers, credentials, raw prompts, and operator personal details with stable placeholders.
- Preserve public-safe anchors only: public DOI strings, public Nostr event IDs if needed, public GitHub URLs if needed, and local artifact paths that do not reveal private infrastructure.
- Use summarized event payloads and daily-log citations rather than raw tool stdout where the raw output may contain private runtime context.
