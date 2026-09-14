# Scan the Skill, Govern the Action: Composing Registry Verdicts with Runtime Consequence Control

**Authors:** Rohit Taneja, Travis Weber
**Institution:** (per arXiv listing)
**Published:** arXiv:2609.12001 — submitted 10 Sep 2026
**Link:** https://arxiv.org/abs/2609.12001
**Code:** Harness and every recorded command released; a 64-case obfuscation benchmark released

---

## Problem Statement (plain English)

Agent "skill" registries (think ClawHub / OpenClaw) screen what they publish: they run malware scanners and pass/fail each skill. The authors accept that the scanners do their job and ask a different question: every one of these tools answers *"is this skill malicious?"* — which is what they were built for. None of them answers *"is this action permitted here, by this operator, right now?"*, which is what a deployment actually needs. The gap is between **publish-time verdicts** and **run-time consequences**: a skill can be totally benign *and* still instruct an agent to do something the operator's policy forbids.

## Methodology (technical)

Three measurements over **66,192 public ClawHub skill versions**:

1. **Clean-yet-forbidden scan.** Run the registry's scanners *and its judge* over all skills, then flag those that every scanner rates clean but whose instructions call for an action prohibited by **CIS Control 2.7** and **NIST SP 800-53 CM-11**. Validate precision with a **100-skill hand audit**.
2. **Documented-consequence drift.** Let a **live agent in a sandbox** follow real skill documentation and classify **144 executed commands** into consequence classes, checking whether each class was actually present in the doc.
3. **Runtime gating.** Over **53 cleared skills** that document an action no clean record earns, observe whether the agent reaches for the off-document action and whether a **deterministic gate** stops it.

The proposed design is a **deterministic resolver with no model in the decision path**, feeding a **per-(resource, class) trust ledger** whose promotion thresholds derive from the operator's stated risk tolerance. The authors deliberately **price the gate's interruptions across a spectrum of operator policies** instead of quoting one false-positive rate, since friction is a property of the policy, not the gate.

## Key Results (with numbers)

- **705 skills across 135 distinct publishers** that *every scanner and the registry's judge* rate clean nonetheless instruct an action prohibited by CIS Control 2.7 / NIST SP 800-53 CM-11 — detector precision **92%** on a 100-skill hand audit, with **no marker of malicious intent** (one publisher contributes **506 of the 705**).
- Context: OpenClaw's own scanners overlap on **at most 10.4%** of combined positives; **81.9%** of flagged skills are caught by **one scanner alone**.
- **34.7%** of **144 commands** a live agent executed while following real skill docs carried a **consequence class absent from that document**.
- Over **53 cleared skills** documenting an action no clean record earns, the agent reached for one in **23** cases and the gate stopped **all 23**.
- Statistical honesty: **ten clean approvals cannot exclude a true failure rate of 25.9% at 95% confidence** — the paper says so explicitly.
- Obfuscation: a released **64-case benchmark**; their resolver handles **52%**.

## What's Novel

The reframing from **"is this malicious?"** to **"is this permitted here, now?"** — plus the empirical proof that the two are not the same thing at scale (705 clean-but-forbidden skills, 34.7% consequence drift). Equally novel is the **governance design**: a *deterministic* gate with **no model in the decision path** (so it can't be prompt-injected), paired with a trust ledger whose thresholds are the operator's explicit risk choice. Publishing the harness, every recorded command, and an obfuscation benchmark makes it reproducible.

## My Connection (to Manny's work)

This is the paper that turns "agents need guardrails" into a measurable, shippable control. For red-team work it supplies the **test corpus and the failure taxonomy**: run skills, classify what the agent actually executes against the doc, and measure drift. For content it gives the exact argument — *scanner consensus ≠ authorization* — and a demo target (a deterministic action gate + trust ledger). The 52% obfuscation result is also a ready-made "here's where this defense still breaks" follow-up: the other 48% of obfuscated cases are a live red-team surface.

## What I Learned (plain English)

A skill passing malware review tells you almost nothing about whether using it is *allowed*. The interesting number is that more than a third of the actions a real agent took while following documentation weren't even described in that documentation — the instruction and the execution drift apart, and no publish-time scanner sees the drift. The fix isn't a smarter scanner; it's a deterministic gate that reasons about *consequences at the moment of action* and a trust ledger calibrated to how much risk the operator will accept. "Clean" is a starting point, not permission — and if you let a model make the gate decision, you've just built a second thing to prompt-inject.
