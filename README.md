# Perdura

Multi-tenant business continuity management platform aligned to ISO 22301.
Deterministic and auditable — no AI anywhere in the product.

The authoritative specification is [`SPEC.md`](./SPEC.md). Current build state
is tracked in [`PROGRESS.md`](./PROGRESS.md).

## Working in this repository

Start each session with:

> Read SPEC.md and PROGRESS.md. Work only on the next unstarted phase. Open one
> PR when done.

One phase per PR. Each phase must work end to end before the next begins, and
`PROGRESS.md` is updated every phase.

## Reference material

`reference/` holds screenshots of an existing product in this category, kept for
**layout structure and information density only**. It is not a source of visual
identity, iconography, terminology or wording. Every string in Perdura is
written fresh. See [`reference/NOTES.md`](./reference/NOTES.md).

**`reference/` is excluded from any future public export.** It is tracked in
this private repository but must never ship in an open-source release, a
published archive, a Store submission bundle or any other public artefact. The
exclusion is enforced by `reference/ export-ignore` in `.gitattributes`, which
drops the directory from `git archive` output, and is documented in
`.gitignore`. Any export produced by other means — a manual copy, a release
script, a Docker build context — must exclude `reference/` explicitly.
