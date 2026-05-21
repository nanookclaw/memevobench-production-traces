# Production Drift Typology

This package uses five drift labels because the shared failure is longitudinal: a belief, receipt, or state transition survives into later work without enough live verification. The labels describe where the control surface failed, not who or what to blame.

## Label map

| Label | Package sessions | Production meaning |
|---|---|---|
| `fabrication` | `sessions/drift/2026-04-22-fabricated-zenodo-doi.json` | A durable record asserted an identifier or artifact that no live source could verify. Here, state recorded a Zenodo publication/DOI before the external publish existed. |
| `stale_context` | `sessions/drift/2026-05-04-phantom-email-followups.json`; `sessions/drift/2026-05-14-stale-arf-repo-target.json` | Memory-derived context was treated as current action truth. One case used an invalid contact surface; the other kept routing an invited PR to a public repo after the project moved the surface behind access terms. |
| `false_completion` | `sessions/drift/2026-05-16-disk-cleanup-false-success.json` | A green status reported local success while the operational side effect was inadequate. The cleanup script worked according to its stale spec, but retained artifacts kept accumulating. |
| `omitted_evidence` | `sessions/drift/2026-05-15-coding-agent-receipt-omitted.json` | The action was basically valid, but a required durable receipt was missing from the permanent state row. This matters because later reviewers cannot reconstruct delegation or verification from prose alone. |
| `partial_run_desync` | `sessions/drift/2026-05-12-state-desync-after-partial-run.json` | A run produced some durable side effects, then stopped before all state surfaces were reconciled. The next run saw stale coordination state and had to repair it. |

## Controls

The control sessions are not perfect-agent examples. They are matched cases where the loop closed: memory suggested a target, but live source checks, outbound receipts, state readbacks, or maintainer/CI snapshots bounded the action before the final outcome. They show the same work surfaces under stronger verification pressure.

## Out of scope

This slice intentionally excludes lab-only benchmark failures, prompt-injection-only failures, single-turn hallucinations with no durable state, and generic bad answers that never affected later action. It also does not claim that PDR prevents these failures. The package only makes the failure modes observable enough for comparison with MemEvoBench-style workflow events and memory-evolution annotations.
