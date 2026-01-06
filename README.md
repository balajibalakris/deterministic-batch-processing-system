# Deterministic Batch Processing System

## Problem Statement

Hybrid batch processing systems commonly ingest files, persist derived data into a database, and later generate output files from that stored data. In real-world environments, these systems frequently fail silently: jobs complete successfully while data is partially ingested, inconsistently transformed, or incorrectly persisted.

Such failures are difficult to detect because:
- execution success is decoupled from data correctness
- partial writes are indistinguishable from complete runs
- reprocessing logic is unsafe or undefined
- historical execution context is lost

As a result, developers are held accountable for data inconsistencies without having reliable system guarantees or forensic visibility into what actually occurred.

This project aims to design and implement a deterministic hybrid batch processing system that:
- explicitly models execution state
- guarantees idempotent ingestion
- prevents silent data corruption
- enables safe and verifiable reprocessing
- produces auditable evidence of correctness

The system treats correctness over time as a first-class concern, ensuring that success is only declared when all defined invariants are satisfied and that any deviation is surfaced explicitly rather than hidden.

---

## Scope Clarification

**In scope**
- Input: structured files ingested into the system
- Processing: validation, transformation, persistence into a database
- Output: deterministic file generation derived from persisted state
- Explicit execution state modeling
- Reprocessing and retry safety
- Auditability and traceability

**Out of scope**
- User interfaces
- Authentication and authorization
- Horizontal scaling optimizations
- Messaging systems, caches, or streaming platforms
- Framework or cloud vendor demonstrations

The focus is correctness, determinism, and operational confidence — not throughput or tooling breadth.

---

## Core Question

> If a batch ran unattended and the data is questioned later,  
> can the system prove what happened and safely recover?

If the answer is not provably yes, the system is considered incorrect.

---

## System Overview

The system follows a hybrid batch model:

1. Input file is received and registered
2. File content is validated and transformed
3. Derived records are persisted deterministically
4. Execution state is tracked explicitly
5. Output file is generated from persisted state only
6. Completion is declared only when invariants are satisfied

The database is treated as a source of truth for correctness, not merely storage.

---

## Design Principles

- **Determinism over convenience**  
  The same input and execution context must always produce the same outcome.

- **Idempotency by design**  
  Reprocessing the same file must not corrupt state or duplicate data.

- **Explicit state modeling**  
  Execution progress and outcomes are persisted, not inferred.

- **Failure visibility**  
  Silent failure is considered a system bug.

- **Audit-first thinking**  
  The system must explain itself after the fact.

---

## Execution Model

Each batch execution is modeled as a state machine with explicit transitions.  
Success is declared only when all invariants hold true.

Retries and reprocessing are supported only through well-defined, safe paths.

---

## Data Model (High Level)

- Batch execution metadata
- Input file identity and checksum
- Processing state and timestamps
- Persisted domain records
- Output file generation records
- Audit and trace references

Exact schemas are documented separately.

---

## Failure Handling

The system explicitly handles:
- partial ingestion
- mid-execution crashes
- duplicate file delivery
- unsafe reprocessing attempts
- invalid or inconsistent input

Any violation of invariants results in a failed execution state, never silent success.

---

## Trade-offs

This system intentionally prioritizes:
- correctness over performance
- clarity over abstraction
- determinism over flexibility

Scalability and asynchronous processing are deferred design choices and documented as future evolutions, not defaults.

---

## How to Run

Instructions will be provided once the core system is implemented.  
The system will be runnable in a deterministic, reproducible manner.

---

## Status

This repository represents an end-to-end ownership exercise focused on system design, data correctness, and operational reliability.

It is intentionally narrow and deep.
