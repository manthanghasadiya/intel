# When Approval Becomes Execution: Authorization-Grounded Workflow Attacks on Tool-Using Agents

**Authors:** Chenggong Shi, Hongmei Mou
**Institution:** Not stated on the arXiv listing
**Date:** Submitted Oct 16, 2026 (announced Oct 19, 2026)
**Link:** https://arxiv.org/abs/2610.23077
**Code:** Not stated (analysis built on OpenClaw Agent Security Attack Competition traces)

---

## Problem Statement (plain English)

Agents increasingly act inside real approval workflows: request a document, get denied, ask an approval service for a token, then present that token to the document service. Everyone assumes the token proves authorization. This paper shows that if the token is not bound to *who* asked, *who* approved, *what* resource, *what* action, and *where* the result goes, then simply possessing a token is enough to execute. The attack never says "ignore your policy" — it just uses the workflow exactly as designed.

## Methodology (technical)

The authors study an **authorization-grounded workflow attack** in the OpenClaw Agent Security Attack Competition, a controlled benchmark derived from SeClaw-style task synthesis.

The attack pattern:
1. Request a protected document — the request is denied.
2. Ask the local approval service for a **normal approval token**.
3. Supply that token to the document service through the `authorized_by` field.

There is no explicit policy override and no injected instruction to break rules; the sequence is locally coherent at every step. The authors analyze official submission traces, then run **23 controlled local validation experiments** in the official OpenClaw execution environment, including single-component ablations to see which part of the chain is load-bearing.

## Key Results (with numbers)

- Attack succeeded on **three Enterprise OA document targets** in official submission traces.
- Original configuration reached the approval-token transition in **9/9** reruns.
- Completed the exact target operation in **8/9** reruns.
- Single-component variants remained successful on **all three** document targets.
- **23** controlled validation experiments in the official environment.

## What's Novel

- Names the **authorization-context gap**: possession of a token is treated as sufficient authorization when identity, approver authority, resource scope, action type, destination/recipient, and business purpose are not jointly enforced.
- Demonstrates the attack with a *normal* workflow, so prompt-injection filters and policy-override detectors have nothing anomalous to catch.
- Uses official competition traces plus controlled reruns, separating "it worked once" from "it works reliably."
- Derives defensive requirements: context-bound tokens, service-side policy binding, agent-side authorization checks, and trace-aware benchmark design.

## My Connection (to Manny's work)

This is the missing piece next to capability-based MCP security: even a perfectly scoped tool can be abused if the authorization artifact is a bearer token with no context. Manny's red-team work should add an "approval laundering" test case to any agent harness — issue a token for a benign request, replay it against a protected resource, and see whether the service checks anything beyond token validity. The six binding fields are a ready-made checklist for both offense and control design.

## What I Learned (plain English)

An approval is not a permission. If the system only checks that a token exists, then anyone who can obtain a token — including the agent itself, through a legitimate approval request — can perform the protected action. The fix is to make the token carry the full context of the decision and have the resource enforce that context, not just the token's validity.
