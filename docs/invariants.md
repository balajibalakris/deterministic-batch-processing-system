# System Invariants 

This document defines non-negotiable invariants for the batch processing system.
These invariants describe what must remain true regardless of execution order,
retries, failures, reprocessing, or human intervention.

The system records **regulatory assertions**, not files, jobs, or executions.

---

## 1. Atomic Fact (Root Axiom)

A reporting authority asserts that a customer breached a regulatory rule within a defined reporting period.

This statement defines the smallest indivisible truth the system records.
All system behavior must preserve the correctness of this assertion.

---

## 2. Identity Invariants

- A logical input is identified by the combination of:
    - reporting authority
    - subject (customer)
    - regulatory rule identifier and version
    - explicit reporting period (start and end)

- Two inputs that assert the same logical fact are the same input,
  regardless of physical representation.

- Differences in physical representation must never create a new identity.

- Corrections, withdrawals, or replacements of an assertion must be expressed
  as explicit new assertions linked to the original identity.

- Silent overwrite of an existing logical fact is forbidden.

---

## 3. Determinism Invariants

- Given persisted state alone, the system must deterministically classify any incoming input as exactly one of:
    - new logical fact
    - duplicate of an existing fact
    - correction of an existing fact
    - conflicting assertion (illegal state)

- Reprocessing the same logical input at any time must produce the same classification outcome.

- The system’s belief about regulatory truth must be reconstructable
  without access to original files or execution history.

---

## 4. Identity Stability Across Time

- Logical identity must remain stable across:
    - retries
    - restarts
    - backfills
    - resubmissions
    - manual reprocessing

- Time is defined by durable state transitions, not by execution duration or wall-clock time.

---

## 5. Explicit Non-Invariants

The following are intentionally excluded from identity and determinism:

- filenames or file paths
- report IDs
- file formats or serialization
- transport or sharing mechanisms
- batch job names or execution timestamps
- scheduling or runtime duration

These elements may exist as metadata or audit attributes but must not influence identity.

---

## 6. Enforcement Principle

If an invariant can be violated silently, it is considered unenforced and therefore invalid.
Any invariant violation must result in an explicit, inspectable failure state.


# Execution Boundary Invariants

These invariants specify when a logical input is acknowledged, committed, or rejected,
independent of execution flow, retries, or job success.

Execution boundaries are defined exclusively by durable state transitions.

---

## 1. Acknowledgement Boundary

- A logical input is not considered **acknowledged** until its logical identity is durably recorded by the system.

- Acknowledgement represents the system accepting responsibility for deciding the input,
  not accepting the truth of the input.

- Validation, correctness checks, or execution success must not determine acknowledgement.

---

## 2. Commitment Boundary

- An assertion is not considered **committed** until its authoritative state is fully and durably persisted.

- Execution completion, job success, or in-memory state must not be treated as commitment.

- Commitment is an irreversible transition that establishes system truth.

---

## 3. Partial State Prohibition

- It must be impossible for a partially committed assertion to be observable to any consumer.

- Either the authoritative assertion state exists in full, or it does not exist at all.

- No downstream logic may rely on intermediate or transient execution states.

---

## 4. Failure Boundary

- Execution success without a committed state transition must not change the system’s belief about any input.

- Execution failure prior to commitment must leave system truth unchanged.

- All failures that prevent commitment must be explicitly representable as a rejection state.

---

## 5 . Boundary Observability

- All execution boundary states must be determinable from persisted state alone.

- Logs, batch job executions, timestamps, or external systems must not be required to determine truth.

If an execution boundary cannot be observed from durable state, it is considered unenforced and invalid.


# State Transition Invariants

- A logical input may exist only in one of the following states:
  - ACKNOWLEDGED
  - COMMITTED
  - REJECTED

- The only allowed state transitions are:
  - ACKNOWLEDGED → COMMITTED
  - ACKNOWLEDGED → REJECTED

- No other state transitions are permitted.

- It must be impossible for a logical input to transition:
  - from COMMITTED to any other state
  - from REJECTED to any other state
  - directly to COMMITTED or REJECTED without first being ACKNOWLEDGED

- State transitions must be monotonic.
  Once a logical input leaves a state, it must never re-enter that state.

- COMMITTED and REJECTED are terminal states and must never be reversed.


# Idempotency Invariants

- Acknowledgement of a logical input must be safe to repeat.

- Applying the same logical input more than once must not create multiple authoritative committed assertions.

- For a given logical identity, at most one committed assertion may exist.

- Commitment of authoritative truth must be a single-occurrence operation and must never be duplicated under retries.

- Idempotency must be enforced at the persistence boundary where authoritative state is written.

- Input ingestion and output generation may be repeated arbitrarily, provided they do not alter committed truth.

# Failure Visibility Invariants

- Failures are acceptable only if they leave the system in a state where no authoritative truth is ambiguous.

- Failure before acknowledgement must leave no logical input recorded.

- Failure after acknowledgement but before commitment must leave the logical input in an acknowledged or explicitly rejected state.

- Failures that occur during or threaten authoritative commitment must be loud and blocking.

- It must be impossible for a failure to result in partially committed or ambiguously committed truth.

- For every failure, the system must persist sufficient information to determine:
  - the affected logical identity (if any)
  - the execution boundary reached
  - whether authoritative truth was committed
  - whether reprocessing is safe or forbidden

---

# Reprocessing Invariants

- Reprocessing is allowed only when it cannot create, alter, or overwrite an existing authoritative committed assertion.

- Reprocessing is permitted when a logical input is in one of the following states:
  - ACKNOWLEDGED but not COMMITTED
  - explicitly REJECTED
  - deterministically classified as duplicate, correction, or conflict

- Reprocessing must never silently mutate or replace committed authoritative truth.

- All reprocessing attempts must resolve to the same logical identity classification given the same persisted state.

---

# Preservation and Immutability

- The following must be preserved across all executions and reprocessing attempts:
  - logical identity
  - acknowledgement records
  - all state transitions
  - classification outcomes (duplicate, correction, conflict, rejection)

- Once a logical identity is acknowledged, it must never change.

- Once an authoritative assertion is COMMITTED or REJECTED, that historical record must be immutable.

- Historical state transitions must never be deleted, overwritten, or rewritten.

---

# Audit & Accountability Invariants

- The system must persist sufficient evidence to reconstruct:
  - what authoritative assertions were made
  - by which reporting authority
  - for which subject
  - under which rule and rule version
  - for which reporting period
  - and what final and historical states they reached

- The system must persist data sufficient to establish:
  - **who** asserted or acted (authority context)
  - **when** each state transition occurred
  - **why** an input was rejected, corrected, or classified as conflicting

- Any information required to justify or defend a decision must be persisted at the time of decision.

- The system must not rely on logs, batch execution history, or external systems to reconstruct authoritative truth.

---

# Human Interaction Invariants

- Operators may:
  - trigger retries or reprocessing
  - inspect system state
  - initiate explicit correction workflows

- Operators must never:
  - directly modify authoritative committed assertions
  - delete or overwrite historical records
  - force state transitions outside defined invariants

- It must be technically impossible to:
  - create duplicate committed assertions for the same logical identity
  - overwrite or bypass authoritative truth
  - resurrect rejected or committed inputs into earlier states

- The system must prevent, not merely warn about:
  - duplicate identity acknowledgement
  - duplicate authoritative assertion commitment
  - silent overwrite of committed truth

---

These invariants will be the guiding principles in building the appilication end to end.
