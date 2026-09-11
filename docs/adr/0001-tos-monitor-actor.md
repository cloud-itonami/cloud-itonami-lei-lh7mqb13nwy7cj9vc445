# ADR-0001: ToSMonitor-LLM ⊣ ToSArchiveGovernor -- a governed actor layered on this archive

- Status: Accepted (2026-07-24)
- Related: [`com-junkawasaki/root` ADR-2607110300](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607110300-cloud-itonami-lei-corporate-tos-catalog.edn)
  (the archive-only design this repo was created under -- unchanged by this
  ADR); [`com-junkawasaki/root` ADR-2607241900](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607241900-cloud-itonami-lei-tos-monitor-actor-pilot.edn)
  (the original 1-repo pilot, on `cloud-itonami-lei-2572ibtt8cczw6au4141`,
  P&G); [`com-junkawasaki/root` ADR-2607242000](https://github.com/com-junkawasaki/root/blob/main/90-docs/adr/2607242000-cloud-itonami-lei-tos-monitor-actor-batch10.edn)
  (the 10-repo validation batch this repo is part of -- this repo is the
  one whose real `:terms-of-use` doc-type surfaced a vocabulary bug in the
  pilot, fixed before this repo was built).

## Context

This repository archives the publicly published Terms of Use of
Brookdale Senior Living Inc., per ADR-2607110300 -- a read-only reference
archive. As part of a 10-repo validation batch extending the
`cloud-itonami-lei-2572ibtt8cczw6au4141` pilot (see ADR-2607241900/
ADR-2607242000 for the full fleet-level design rationale), this repo gains
a governed actor layer on top of the unchanged archive.

## Decision

Identical design and code to the pilot and every other repo in this batch
(`src/tosmonitor/{governor,phase,operation,registry,advisor}.cljc` are
byte-for-byte identical across all of them) -- see ADR-2607241900 for the
full rationale of each of the six HARD governor checks, the single
always-escalate `:tos/change-proposal` actuation, and the mock-advisor-only
scope. Only `tosmonitor.store`'s company/baseline demo data is specific to
this repo:

- **Company**: Brookdale Senior Living Inc., LEI LH7MQB13NWY7CJ9VC445,
  website `https://www.brookdale.com`.
- **Baseline provenance** (real, from this repo's own `80-data/public/
  tos.journal.edn`): source-url
  `https://www.brookdale.com/en/privacy-policy/terms-of-use.html`,
  retrieved-at `2026-07-10`, doc-type `:terms-of-use`.
- **Baseline full text**: a short, hand-written representative excerpt (not
  the real archived page), with a self-consistent SHA-256 computed from
  that excerpt itself -- matching the pilot's own convention
  (ADR-2607241900).

**This repo's real archive data (`:terms-of-use`, not `:terms-of-service`)
is the exact case that surfaced ADR-2607242000's Decision 3**: a grep
survey across all 155 `cloud-itonami-lei-*` journal files, prompted by
building this batch, found `tosmonitor.registry/known-doc-types` in the
pilot had been a GUESSED set that did not include `:terms-of-use` (or
several other real archive doc-types). That was fixed in the pilot repo
(a separate small PR, re-tested green) before this repo was built, so this
repo ships with the corrected vocabulary from the start and its own
`kbb -M:dev:test`/`kbb -M:dev:run` correctly treat its real
`:terms-of-use` doc-type as valid, not a HARD hold.

The archive-of-record (`80-data/public/tos.journal.edn`) is never touched;
`commit-record!` only writes to this actor's own Store.

## Consequences

Same as the pilot (ADR-2607241900) and the batch (ADR-2607242000), plus:
this repo is the concrete evidence for why the batch-of-10 validation was
worth doing before any fleet-wide rollout -- a 1-repo pilot alone could not
have surfaced a doc-type vocabulary bug that only a second, genuinely
different company's real data revealed.

## Run

```bash
kbb -M:dev:run     # walk a clean lifecycle + all six HARD-hold checks + a phase-0 hold + a backend swap
kbb -M:dev:test    # governor contract · phase invariants · store parity · advisor smoke
kbb -M:lint        # clj-kondo (errors fail; CI mirrors this)
```
