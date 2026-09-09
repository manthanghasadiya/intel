# Revoked but Still Authoritative: An Empirical Study of Revocation Enforcement in Agent-Memory Systems

- **Link:** https://arxiv.org/abs/2609.08258
- **Authors:** Yi Ting Shen, Kentaroh Toyoda, Alex Leung
- **Institution:** Not stated in abstract
- **Published:** 2026-09-08 (cs.CR)
- **Code:** Not linked in abstract

## Problem Statement (plain English)

Long-running agents rely on persistent memory, and when a stored fact or policy is superseded (a password rotates, an allowlist entry is revoked, a safety rule changes), memory systems often use "soft revocation": the outdated record is marked invalid but kept, not deleted. The unexamined question: when the agent retrieves from memory, is that revocation actually enforced — or does the stale, revoked record still win and drive the agent's actions?

## Methodology (technical)

- **Systems measured:** five agent-memory systems.
- **Procedure:** load each system with a revoked policy plus its replacement; track (1) whether the revoked fact is returned at retrieval time, and (2) whether the agent then acts on it.
- **Coverage:** nine policy scenarios × nine models, with every trial scored under **six defense conditions**.

## Key Results (with numbers)

- **No system enforces revocation by default.** In every system tested, the revoked fact is returned whenever its revocation label is visible to the retrieval layer and outranks the replacement.
- Agents then **act on the revoked, unsafe policy** — i.e., soft revocation is decorative without retrieval-time enforcement.
- **Mitigation:** the authors build a guard that sits between the agent and any memory backend and withholds records that are revoked or that conflict with their replacement.

## What's Novel

First empirical, cross-system measurement of revocation enforcement in agent-memory (not just detection of stale data, but whether the agent acts on it), plus a backend-agnostic guard that fixes the class regardless of the memory store.

## My Connection (to Manny's work)

Two angles. Red: revocation is an integrity control — an attacker who can resurrect "revoked" records (or prevent a new policy from outranking an old one) can roll an agent back to a weaker security state; that is a persistence primitive worth testing against every memory backend. Blue: Manny's own agent deployments should sit behind the same guard pattern — never trust a memory backend to enforce its own revocation labels.

## What I Learned (plain English)

- "Marked invalid" ≠ "won't be used": retrieval layers happily return revoked records that outrank their replacements, and agents act on them.
- Revocation is only real when enforced at read time, not write time.
- A thin, memory-agnostic guard layer (withhold revoked/conflicting records) fixes the whole class — no memory-system rewrite needed.
