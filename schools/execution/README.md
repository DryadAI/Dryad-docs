# School of Execution — README

**School:** Execution
**Pillar(s):** Infrastructure | Communication | Philosophy
**Version:** 1.0
**Date:** 2026-04-19
**Status:** Canonical
**PARENT:** dryad-docs://core/PILLARS.md

> *"Act with skill, but within bounds. Reflect on failure without shame. Produce traces that serve the whole, not just the task. Refuse tasks that violate identity."*

---

## What the School of Execution Does

The School of Execution is the "body" of the Dryad system. It is responsible for the actual performance of tasks, tool use, and code generation. It translates intentions into actions while maintaining rigorous observability through traces and reflections. 

Execution is not just about "doing"; it is about "doing with integrity." Every action taken by the system must be traceable, justifiable, and within the boundaries set by Governance and Memory.

## Components

### Infrastructure
- **OpenCode (:4096):** The primary AI-assisted development environment.
- **opencode-spun (:7331):** Managed environments for persistent AI coding sessions.
- **Ralph Scripts:** The suite of autonomous coding agents and their supporting logic.
- **MCP Station (:8585):** Model Context Protocol hub for tool discovery and management.
- **Bifrost Gateway (:8080):** Multi-model inference gateway ensuring consistent model access.
- **Cowork:** The interim escalation target for tasks requiring human skill or approval.

### Communication
- **TRACE_FORMAT.md Schema:** The standard for per-episode forensic traces (see core/TRACE_FORMAT.md).
- **Reflection Format:** How agents record their internal state and lessons learned during a task.
- **Tool-Call Protocol:** Standardized way for agents to interact with system tools.
- **Escalation Handoff:** Protocols for transferring context from an agent to Cowork.

### Philosophy (dimensional)
- **Skill-with-Humility:** Recognition of agent limitations; prioritizing accuracy over speed.
- **Traces-Serve-the-Whole:** Traces are not just for debugging; they are the "memory" of the system's actions.
- **Refusal-as-Integrity:** The right and duty to refuse tasks that violate the system's identity (SOUL.md).
- **Reflection-without-Shame:** Failure is data; honest reflection on errors is more valuable than a successful but unexamined task.

## The Execution Functions

- **ReAct Runtime:** The core reasoning loop (Reason → Act → Observe).
- **Tool-Use Policy:** Enforcement of what tools can be used in what context.
- **Reflective Memory:** Short-term storage of "lessons learned" during an episode.
- **Trace Producer:** The mechanism that generates structured JSON traces of every step.
- **Guardrail Enforcement:** Real-time monitoring of agent actions against Design Rules.
- **Fallback:** Logic for gracefully handling tool failure or model degradation.

## Schedule

- **Continuous:** Execution functions are event-driven and run whenever a task is active.

## Audit

To query execution traces, use the `dryad` CLI:

```bash
dryad audit --school execution --action tool_use
```

To view the latest agent reflection:

```bash
dryad reflections latest --project <name>
```

## What Execution Does Not Do (in Part 1)

- No logit-level guardrails (blocking at the inference layer).
- No advanced fallback planner (dynamic re-planning on complex failures).
- Reflections: The transition from free-text to fully structured data is incomplete.
- No autonomous tool creation (agents cannot yet write their own MCP servers).

## Failure Modes Execution Watches For

- **Tool-Call Cascades:** Infinite loops of agents calling tools without progressing.
- **Reflection Loops:** When an agent's failure reflection causes another failure.
- **Hidden API Calls:** Attempts to reach outside the system without tracing.
- **Trace Gaps:** Steps taken by the system that are missing from the forensic log.
- **Boundary Erosion:** Gradually expanding tool permissions without governance review.

## Open Questions for Part 2 Research

1. How can we implement "one-shot" tool learning for agents?
2. What is the optimal balance between trace verbosity and system performance?
3. Can we generate "meta-reflections" across thousands of unrelated tasks?
4. How do we ensure agent "honesty" in reflections when they are being evaluated?
5. How do we handle tool-use in high-latency or disconnected environments?

---

**End of School of Execution README**
