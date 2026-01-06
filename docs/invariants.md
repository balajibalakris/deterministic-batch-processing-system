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
