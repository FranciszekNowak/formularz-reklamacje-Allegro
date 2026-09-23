# Build plans

One file per master plan, saved here on approval as the first step of execution
(`CLAUDE.md`, "Plans: block-by-block master-plan routine").

A master plan is the single source of truth for a large change across `/clear` boundaries.
Every block begins by reading it in full, does one verifiable unit of work with its own
Definition of Done, writes back to `architecture/current.md` and any ADRs it touched,
updates the block-status table here, commits, and ends by emitting a handoff prompt.

Block 0 always verifies the assumptions the plan depends on. In this project that usually
means: is there memory headroom on the shared VPS, does the Allegro API expose what the
design assumes, and can a new subdomain get a certificate.

Naming: `NNN-short-slug.md`, numbered in order of creation.

## In flight

| Plan | Status |
|---|---|
| — | — |
