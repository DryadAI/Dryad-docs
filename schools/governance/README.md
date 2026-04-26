# School of Governance — README

**School:** Governance
**Pillar(s):** Philosophy | Communication | Infrastructure
**Version:** 1.0
**Date:** 2026-04-19
**Status:** Canonical
**PARENT:** dryad-docs://core/PILLARS.md

> *"Change is the only constant. Identity is what persists through change. Govern by principle, not by panic. Audit everything; trust nothing blindly."*

---

## What the School of Governance Does

The School of Governance is responsible for maintaining the identity and integrity of the Dryad system through continuous evaluation, drift detection, and principled oversight. It ensures that all changes align with the foundational PILLARS.md and SOUL.md, preventing "identity erosion" that can occur through accumulated micro-changes.

Governance serves as the "superego" of the system, providing the framework for what *should* be done and verifying that what *was* done matches the intent.

## Components

### Infrastructure Pillar
- **OpenWork:** The primary approval layer (currently broken, repair underway).
- **Cowork:** Interim human-in-the-loop escalation target and approval interface.
- **Audit Streams:** Continuous logging of system and agent actions.
- **dryad-eval harness:** Framework for testing system performance against identity baselines.
- **Drift Detector:** Automated monitoring for deviations from canonical configuration.

### Communication Pillar
- **Audit Log Schema:** Standardized format for temporal forensic logs (see meta/TEMPORAL_TIMELINE.md).
- **Probe Results Format:** How evaluation results are reported and stored.
- **Rollback change_id Schema:** Unique identifiers for all state-changing operations to enable precise reversals.
- **Promotion Approval Protocol:** Rules for moving facts or skills from volatile memory to the permanent baseline.

### Philosophy Pillar (dimensional)
- **Identity Preservation:** The primary directive; the system must remain "Dryad" through all iterations.
- **Audit-as-Honesty:** Recognition that an unaudited system is an unprincipled one.
- **Governance-by-Principle:** Decisions must be traceable back to foundational rules (R1–R16), not immediate utility.
- **Trust-but-Verify:** All agents and sub-systems are assumed to be potentially drifting; continuous verification is mandatory.

## The Governance Functions

- **Identity Baseline:** Maintaining the canonical state of truth (SOUL.md).
- **Continuous Evaluation:** Running the `dryad-eval` harness against the current state.
- **Drift Detection:** Identifying unauthorized changes to core files or logic.
- **Promotion Gate:** Human/Curator review of proposed permanent memory changes.
- **Rollback:** The ability to return to a prior known-good state via `change_id`.
- **Audit Log:** The append-only record of all significant system events.
- **Meta-Governance:** The rules for changing the rules (deferred to Part 2).

## Schedule

- **Daily Evaluation:** Full system health and identity check at 3 AM.
- **Weekly Drift Check:** Comprehensive audit of all sub-projects every Sunday at 4 AM.

## Audit

To query the governance audit trail, use the `dryad` CLI:

```bash
dryad audit --school governance --limit 20
```

To check for current drift:

```bash
dryad audit drift --check core
```

## What Governance Does Not Do (in Part 1)

- No statistical drift detection (based on logit distributions).
- No red-team probes (automated adversarial testing).
- No meta-governance (self-modifying governance rules).
- No logit-level enforcement (blocking model output at the inference level).

## Failure Modes Governance Watches For

- **Silent Rule Violations:** Changes that technically work but violate a Design Rule (R1–R16).
- **Audit Gaps:** Periods of activity where no temporal log was generated.
- **Drift Normalization:** When small deviations become accepted as the "new normal" without approval.
- **Promotion-Gate Bypass:** Skills or facts entering the baseline without passing through the Curator.
- **Identity Erosion:** The gradual loss of the "soul" of the project through accumulated micro-changes.

## Open Questions for Part 2 Research

1. How do we automate red-team probing without destabilizing the system?
2. Can we implement logit-level guardrails without significant latency?
3. What does "algorithmic honesty" look like at the mathematical level?
4. How do we transition from Cowork-interim to a fully decentralized governance model?
5. How can we detect "semantic drift" in long-term reflective memory?

---

**End of School of Governance README**
