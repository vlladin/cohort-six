# Adding Era File Support in Lodestar

Implement support for `.era` files in Lodestar to enable efficient long-term storage and distribution of Ethereum consensus data.

## Motivation

Lodestar currently supports consensus layer operations but does not yet natively integrate with the `e2store` / `era` archival format. Era files are specialized `.e2s` files that bundle states and blocks in fixed-size groups (one per `SLOTS_PER_HISTORICAL_ROOT`). They provide a canonical cold-storage representation of Ethereum history, optimized for reading, validation, and checkpoint sync.

Native Lodestar support for `.era` files unlocks several benefits:
- **Historical sync efficiency**: Ability to bootstrap from finalized checkpoints using pre-verified era files instead of downloading full block/state histories.
- **Validator friendliness**: Era files allow lightweight devices to access verified history without recomputation.
- **Interoperability**: Ensures Lodestar can consume and produce files compatible with other clients like Nimbus, enabling ecosystem-wide standardization.
- **Auditability and distribution**: Researchers, light clients, and custodians can rely on `.era` files for fast consistency checks.

## Project description

The project involves implementing reading, verification, and  writing of `.era` files in Lodestar.

### Scope

- **Parsing `.e2s` records**: Implement utilities to read the 8-byte headers .
- **Era grouping**: Add logic to parse `.era` file groups, which include a set of blocks, an era-ending state, and trailing indices.
- **Verification**:
  - Ensure block roots match those recorded in the state.
  - Verify consistency of indices and offsets.
  - Confirm historical root correctness and alignment with checkpoints.
- **Integration with Lodestar sync**:
  - Allow importing an era file as a sync source.
  - Enable checkpoint sync from an era state root.
  - Provide APIs to iterate over blocks and states in an era file.

### Deliverables

- A Lodestar module (`packages/era`) for `.e2s`/`.era` parsing and verification.
- Utilities for inspecting and validating era files from CLI.
- Hooks in Lodestar sync to use `.era` files as data sources.

## Work during the fellowship

Planned steps:
1. Implement reading and writing of `.era` files.
2. Build CLI tools for creating, inspecting, and validating era files.
3. Integrate era support into Lodestar (sync, checkpoint, and processing pipelines).

## Roadmap

1. **Week 8–10** – Implement reading and writing of `.era` files.
2. **Week 11–14** – Implement group parsing and slot index support.Integrate era support into Lodestar .
4. **Week 15** – Build CLI tools for creating, inspecting, and validating era files; add tests.
5. **Week 16+** – Performance tuning .

## Possible challenges 


- Performance tuning.
- Ensuring interoperability with Nimbus and other client outputs.

## Goal of the project

The main goal is to enable Lodestar to **consume and verify `.era` files **. Success would mean:
- Lodestar can parse, validate, and sync from `.era` files.
- Era files become a practical medium for checkpoint sync and historical lookups.
- Lodestar contributes to cross-client adoption of the era archival format.

This work will prepare Lodestar for Ethereum’s history expiry roadmap.

## Collaborators

### Fellows
- [Rahul Guha](https://github.com/guha-rahul)

### Mentors
- 


## Resources

 - [EthResearch: Era archival files for block and consensus data](https://ethresear.ch/t/era-archival-files-for-block-and-consensus-data/13526)
 - [Geth docs: Downloader](https://geth.ethereum.org/docs/fundamentals/downloadera)
 - [Nimbus Guide: Era store](https://nimbus.guide/era-store.html)
 - [HackMD: Era archival notes](https://hackmd.io/@advaita/BkMvD9Qllg)
 - [EtherWorld: History expiry moves forward in Ethereum’s Fusaka upgrade](https://etherworld.co/2025/06/16/history-expiry-moves-forward-in-ethereums-fusaka-upgrade/)
