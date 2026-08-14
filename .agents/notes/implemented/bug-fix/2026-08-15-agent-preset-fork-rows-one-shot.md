# Agent Note: The CLI agent presets bind fork delegation one-shot

Status: implemented

English | [中文](2026-08-15-agent-preset-fork-rows-one-shot.zh.md)

## Problem

[Forked children stay one-shot](../architecture/2026-08-10-fork-children-stay-one-shot.md) binds every shipped fork delegation tool to `backgroundMode: one-shot`, because a continuable child's `report` tool schema and `tool:report` prompt section sit in the request head, ahead of the inherited history, and cancel the provider-side prefix reuse that is fork's only payoff. The change that landed the decision realigned [the base bundle](../../../../packages/bundle/base/cordis.patch.yml) and the two runnable examples; the three CLI agent presets — [standard](../../../../apps/cli/config/agent-presets/standard/agent.cordis.yml), [code](../../../../apps/cli/config/agent-presets/code/agent.cordis.yml), and [cordis](../../../../apps/cli/config/agent-presets/cordis/agent.cordis.yml) — predate the decision, reproduced the base bundle's then-continuable rows at creation, and were not swept.

The presets are the default Web deployment's agent plane, not a side composition: [the web-app bundle](../../../../packages/bundle/web-app/cordis.patch.yml) disables the base delegation-tool rows, mounts each session's agent from a preset defaulting to `standard`, and keeps the base `tool-subagent-report` row on the host plane. Every default Web session therefore offered a continuable `subagent_fork` whose children carried both request-head deltas — the composition the decision rejects as paying fork's duplication cost and collecting none of its benefit. This is the owning note's accepted risk realized: the constraint lives in configuration with no gate, so the stale rows loaded cleanly and nothing failed loud.

## Decision

All three presets bind `subagent_fork` to `backgroundMode: one-shot`, each row carrying the base bundle's comment naming the reason and the owning note. `run_in_background` stays available, following the base bundle rather than the examples' `enableRunInBackground: false`: the presets mount the `job_*` controls over the host-plane background-job registry, so a one-shot background start has its collection path.

The owning architecture note remains the decision's home; its shipped-composition inventory and configuration-file count now include the presets.

## Alternatives considered

**Keep the presets continuable and amend the decision.** The rows carry no recorded intent — they predate the decision and reproduce the base bundle's earlier state — and the decision's analysis applies to these compositions with full force: they run beside the host-plane report registration, so a continuable forked child loses the entire shared span. That is the "ship continuable and accept the loss" alternative the owning note already rejects.

**Add the load-time rejection this regression argues for.** The owning note rejects it because `tool-subagent` cannot observe the report package and the pair is legitimate without it; one realized instance does not change that analysis. Reopening the question would supersede the owning note's accepted risk — a separate decision, not part of this realignment.

**Set `enableRunInBackground: false` as the examples do.** Wrong shape here: the examples disable background starts because their compositions mount no job collection path; the preset compositions do.

## Consequences

- Every preset composition — including the default Web deployment — binds `subagent_fork` one-shot: its schema carries the one-shot wording, `send_message` addresses only spawned children, and a forked child's request prefix again matches its parent's.
- No snapshot fixture changes: no keyless snapshot pins a preset composition's tool schemas, and [the preset catalog e2e](../../../../apps/cli/tests/web-agent-presets.e2e.ts) asserts tool names, which this change keeps. The preset compositions' model-visible schemas stay unpinned by any keyless snapshot; the examples' sidecars pin the one-shot wording only for their own compositions.
