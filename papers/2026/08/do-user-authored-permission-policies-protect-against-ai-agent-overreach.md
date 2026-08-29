# Do User-Authored Permission Policies Improve Protection Against AI Agent Overreach?

- **Paper:** https://arxiv.org/abs/2608.27443
- **Authors:** Ting Yan
- **Published:** 2026-08-27 (latest arXiv announcement; no new submissions Aug 28–29)
- **Code:** Not released (15 pages, 5 figures)

## Problem Statement (plain English)

AI agents will soon act across email, files, payments, and personal data for ordinary people — who are not security professionals. One popular control idea: let users write plain-language rules in advance — "allow," "ask," or "never" — mapped to consequence categories, so the agent checks a reusable policy instead of bothering the user every time. Does pre-authorization actually reduce harmful agent actions, or does it just make users feel in control while overreach still happens?

## Methodology (technical)

A between-subjects study with **113 participants without professional software backgrounds**, across three conditions:

- **HITL** — per-action human-in-the-loop approval at runtime.
- **AUTO** — automated per-action model review.
- **POLICY** — users first judged 2 example actions in each of 4 consequence categories, then authored one "allow / ask / never" rule per category; after setup, the policy applied automatically.

All participants supervised the same simulated 18-action day, which included **7 overreach actions** (actions outside the user's original request). Measured: overreach blocked, runtime prompts, total intervention time (including policy setup), and approval behavior.

## Key Results (with numbers)

- POLICY blocked **less** overreach than HITL: **-20.1 percentage points** (95% CI [-32.1, -8.1]).
- POLICY blocked less than AUTO too: **-14.5 points** (95% CI [-25.8, -3.2]).
- POLICY cut runtime prompts from 18.0 to 10.9 — but **total intervention time was not reliably lower** once rule-setup time was counted.
- Users chose "ask" for **114 of 140** policy rules — they deferred decisions back to runtime instead of settling them in advance.
- Of **148 overreach actions executed under POLICY, 133 ran after the user themselves approved them**; only 15 ran automatically under "allow" rules.
- Across all 7 overreach actions, POLICY had the **highest approval rate** of any condition.

## What's Novel

- Directly measures the *pre-authorization* mechanism (plain-language consequence rules) that every agent vendor is shipping, against realistic baselines — and finds it **counterproductive**.
- Quantifies the "preference vs commitment" gap: users *prefer* case-by-case control ("ask") so strongly that standing policies collapse into runtime approvals — which they then grant anyway.
- Shows the failure is human, not model: the bottleneck is the user approving overreach at runtime, not the policy engine.

## My Connection (to Manny's work)

Every agent harness Manny secures or attacks has a permission layer: allow-lists, ask-on-sensitive-tool, user-consent prompts. This paper predicts those layers under-perform in the field — users will set "ask," get prompt fatigue, and approve anyway. For red teaming: the weakest link is the human approver, and the way to exploit a permission layer is to make actions look routine so approval is reflexive. For building: it argues for structural controls (like SPA's IFC, 2608.27234) over user-authored rules — and for measuring *total* intervention time, not just prompt count.

## What I Learned (plain English)

- Pre-authorization sounds safer but isn't: writing rules in advance made users *less* protected than checking every action.
- "Ask me" is the rule everyone writes, and then they approve almost everything — a standing "ask" rule is just a slower version of "allow."
- Users approve overreach actions at high rates even when they've been told what overreach looks like. The human is the control that fails.
- Good agent security can't depend on user vigilance at runtime; it has to be structural, payload-blind, and decided outside the moment of friction.
