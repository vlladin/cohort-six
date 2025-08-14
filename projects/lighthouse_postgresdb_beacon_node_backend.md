# Lighthouse: Add PostgresDB as an Optional DB Backend for the Beacon Node

## Tagline

Resilient and concurrent-friendly: Enabling PostgresDB support in Lighthouse to improve reliability in NFS environments and broaden database backend compatibility.

## Motivation

The Lighthouse beacon node currently supports two database backends: LevelDB and Redb. While LevelDB offers high performance, it suffers from corruption issues when used over network file systems (NFS), largely due to its lack of support for concurrent access and poor handling of I/O inconsistencies. Redb, while safer, is still not battle-tested in distributed or concurrent environments.

PostgresDB, being a mature, ACID-compliant relational database with robust support for concurrent access and network reliability, offers a promising alternative. Adding support for PostgresDB as a third backend enhances Lighthouse’s resilience, especially in deployments where NFS or remote disks are unavoidable. This work also opens doors for future metrics, observability, and SQL-based tooling integrations.

## Project Description

This project implements PostgresDB as a fully functional and asynchronous key-value store backend for Lighthouse’s Beacon Node. It leverages the existing `KeyValueStore` (and experimental `AsyncKeyValueStore`) abstractions in Lighthouse, implementing all required methods using `sqlx` and `async_trait` to interact with a PostgreSQL database.

The Postgres implementation supports all database columns used by Lighthouse and is integrated behind a cargo feature flag (`postgres`). It also includes a test suite that runs Lighthouse's existing database tests against a live Postgres instance, ensuring correctness and performance stability.

## Specification

* Backend Implementation:
  - Implement the `AsyncKeyValueStore<E>` trait using `sqlx::PgPool` and Asynchronous Rust patterns.
  - Support full DB operations that are core to lighthouse: `get_bytes, put_bytes, key_exists, key_delete, do_atomically, compact, iter_column_keys_from, iter_column_keys, iter_column_from, compact_column, delete_batch, and delete_if`

* Database Schema:
  - This will maintain a table for each `DBColumn` to allow modular, column-based storage while still leveraging Postgres features for indexing and querying.
Each table will have the following base structure
```
CREATE TABLE <column_name> (
    key   BYTEA PRIMARY KEY,
    value BYTEA NOT NULL
);
```
Where possible, column values will use structured Postgres types (e.g., `BIGINT, TEXT, JSONB`) instead of storing all data as `BYTEA`, with a defined Rust–Postgres type mapping for each `DBColumn`. Certain datasets such as state summaries or block metadata may use relational tables with joins, foreign keys, and targeted indexes to improve consistency and query performance. All tables will have a primary key index on `key`, with additional or composite indexes on frequently queried fields (e.g., slot, validator index). The backend will support `do_atomically` for batched CRUD operations within transactions, and iteration functions (`iter_column_keys, iter_column_keys_from, iter_column_from`) will be implemented via SQL cursors or range queries to align with Lighthouse’s iterator-based workflows while leveraging Postgres’s optimization capabilities. As a consideration, prior testing with SQLite in the Slasher DB showed that primary/foreign key constraints degraded batch write performance, so we will evaluate the trade-offs for the beacon node context before committing to such constraints.

* Integration:
  - Add `#[cfg(feature = "postgres")]` support to Lighthouse's database interface layer.
  - Modify configuration options to enable Postgres backend selection via `CLI` or `toml`.

* Testing:
  - Add a dedicated test suite using `tokio::test` to run database tests on a live PostgresDB instance via Docker or local instance.
  - Integrate with existing lighthouse test suite to run with Postgres db backend.

## Roadmap

### * Month 1
You can follow some of the work already done in this [PR](https://github.com/sigp/lighthouse/pull/7685).

* Design Postgres schema and implement foundational methods: `get_bytes, put_bytes, and key_exists`.
* Begin do_atomically implementation and handle executor-related trait issues with `sqlx`.
* Set up local test harness using a live Postgres instance.
Initial test coverage for basic operations

### * Month 2

* Complete implementation of `do_atomically`, including full transaction support.
* Implement iteration support for Postgres-backed column scans.
Wire the Postgres backend into Lighthouse's database abstration (`interface.rs`).
* Introduce cargo feature flag support for postgres.
* Begin writing full backend integration test.

### * Month 3

* Expand test coverage to include edge cases and concurrent writes.
* Benchmark PostgresDB performace relative to levelDB and ReDB.
* Add CLI or configuration support for selecting Postgres as the database.
* Finalize documentation for internal API usage and schema assumptions.

### * Month 4

* Submit implementation PR to Lighthouse repo.
* Address review feedback and perform necessary refactors.
* Ensure CI compatibility, possible using Docker for Postgres testing
* Engage in community testing or early adopter feedback (if available).

### * Month 5

* Finalize and merge the feature into Lighthouse mainline (if accepted).
* Polish documentation and developer notes for future maintainers.
* Share performance notes and NFS reliability testing results.
* Explore additional enhancements (e.g., Postgres connection pooling tuning, advanced metrics support).

## Possible Challenges

### Possible Challenges
* Trait Compatibility: Rust's trait system and `sqlx` executor trait bounds may require complex type juggling, especially for generic functions.
* Atomic Batch Handling: Translating Lighthouse's batch sematics to Postgres transactions may need careful mapping to ensure consistency.
* Performance: Ensuring Postgres matches or exceeds existing backend perfomances (LevelDB & Redb) for standard use cases may require index tuning or write optimizations.
* Test Environment: Ensuring CI support and reliability when running Postgres-based tests (especially with Dockerized DBs).


## Goal of the Project

A successful implementation will:

* Allow lighthouse to run seamlessly using a Postgres backend with no loss of functionality.
* Pass all existing database backend tests using Postgres.
* Provide resilience in NFS-hosted environments where LevelDB and ReDB have limitations.
* Lay the foundation for future enhancements such as analytics, better observability, and external tooling integrations using SQL.

## Collaborators
### Fellow:

[Owanikin](https://github.com/owanikin)

### Mentors:

[UncleBill](https://github.com/eserilev)
