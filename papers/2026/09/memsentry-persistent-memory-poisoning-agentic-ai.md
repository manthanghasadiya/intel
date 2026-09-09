# MemSentry: A Framework for Detecting Persistent Memory Poisoning in Agentic AI

- **Link:** https://arxiv.org/abs/2609.08747
- **Authors:** Ayan Roy, Kaustuvi Basu
- **Institution:** Not stated in abstract
- **Published:** 2026-09-08 (cs.CR)
- **Code:** Not linked in abstract

## Problem Statement (plain English)

Agents with persistent memory can be attacked without touching model weights or system prompts: adversarially crafted content gets written into long-term memory and later steers the agent — suppressing security alerts, enabling privilege escalation, altering trust relationships, or overriding security policies. Nobody has a principled way to decide, at write time, whether a proposed memory write is safe.

## Methodology (technical)

- **MemSentry:** a formal, configuration-driven framework that intercepts every proposed persistent-memory write and returns a deterministic decision: **Accept, Review, or Quarantine**.
- Each write is scored by jointly considering five signals:
  1. **source trust** (who/what produced the content),
  2. **semantic risk** (pluggable classifier — treated as a component, not the contribution),
  3. **attack radius** over a component-dependency DAG (what else could this write influence),
  4. **access risk** (user/asset exposure via an access-control matrix),
  5. **signed security-state delta** (does the operation weaken or strengthen the security posture).
- **Evaluation:** 20-asset random dependency DAG, 10×20 user access-control matrix, 1,000 GPT-4-generated scenarios with a stratified 70/30 train/test split; four semantic classifiers compared: Regex, TF-IDF+SVM, SBERT+LR, SetFit.

## Key Results (with numbers)

- **SBERT+LR is best overall: 91.7% accuracy, 0.908 macro-F1.**
- **All four methods detect 100% of external quarantine-class threats.**
- Verified insiders (max source trust, T=1) are not automatically quarantined — a deliberate, documented boundary: trusted sources get Review rather than Quarantine, trading strictness for usability.

## What's Novel

First framework to make memory-write vetting **deterministic and configuration-driven** (vs. pure LLM-judge vibes), with an explicit security-state-delta model — deciding whether a write strengthens or weakens the system — plus formal attack-radius computation over a dependency graph. Treats the semantic classifier as pluggable, so the policy core is model-agnostic.

## My Connection (to Manny's work)

Memory poisoning is the agent equivalent of persistence: an attacker who can write to long-term memory owns the next N sessions. MemSentry's five signals are a ready-made checklist for hardening any agent Manny operates (Hermes-style memory, MCP context, tool caches) and for designing the red-team test: can planted content survive a write-time filter and steer later actions? The 100% external-threat detection vs. trusted-insider gap also defines exactly where an insider/compromised-source attack should be aimed.

## What I Learned (plain English)

- Memory writes are a security decision point — you can gate them deterministically with source trust + reach + security delta, without relying on the model to behave.
- The weak spot is trusted sources: anyone who already has write access at high trust can still poison memory (only Review, not Quarantine).
- Simple semantic classifiers (SBERT+LR) are enough to catch external poisoning at high recall — you don't need a frontier model to police memory writes.
