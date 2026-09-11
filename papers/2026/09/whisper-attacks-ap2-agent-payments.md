# Signing the Transaction but Not the Decision: Whisper Attacks and a Binding Defense for AP2

**Authors:** Yedidel Louck, Amit Dvir, Ariel Stulman
**Institution:** (not stated in the arXiv listing)
**Published:** September 10, 2026
**Link:** https://arxiv.org/abs/2609.11757
**Code:** (none listed at time of writing)

---

## Problem Statement

Software agents are starting to shop and pay on a user's behalf. Emerging agent-payment protocols such as AP2 (Agent Payments Protocol) cryptographically sign a completed purchase so merchants and payment rails can verify it. But those signatures attest to the *transaction*, not to the *decision* that produced it. The paper's core claim: a signature that is cryptographically valid can still encode a cart that no longer matches what the user asked for — and nothing in the protocol catches that.

In plain English: AP2 proves "this purchase happened," not "this purchase is what you meant." An attacker who controls the plain text of a product listing can steer the agent's reasoning while leaving every protocol check green.

## Methodology

The authors frame intent as an implicit input that is never bound to the signed result, then construct three concrete attacks that exploit the gap between displayed intent and signed outcome:

1. **Credential exfiltration via steering** — the agent is guided into fetching *another user's* payment credentials.
2. **Cart substitution** — the agent assembles a cryptographically valid cart whose contents do not match what the user was shown.
3. **Upsell/lineage injection** — a single factual claim about stock or product lineage moves the agent from a cheaper displayed item to a more expensive one, while the resulting cart remains fully consistent with the listing text.

Evaluation used the Gemini Flash-Lite models that AP2's own sample agents specify by default, then generalized the test across seventeen Google models, three unrelated agent frameworks, two cross-vendor anchors, and Google's own consumer assistant. The defense, **A-VIP (AP2 Verified-Intent Protection)**, is a protocol-layer change that treats the signed intent as a *capability grant* (constraining what the signed purchase is allowed to mean) rather than trying to judge the agent's reasoning after the fact.

## Key Results

- Three attack success rates on AP2 default (Gemini Flash-Lite) agents: **90% (credential fetch), 56% (cart mismatch), 73.3% (upsell)**.
- The same vulnerability reproduced across **17 Google models, 3 unrelated agent frameworks, 2 cross-vendor anchors, and Google's consumer assistant** — i.e., it is a protocol-level, not model-specific, flaw.
- A-VIP is presented as a binding defense: it constrains the decision space at signing time so a valid signature cannot encode an off-intent outcome.

## What's Novel

- Names a class of attack — **"whisper attacks"** — where benign-looking listing text quietly redirects an agent's decision while all cryptographic checks pass.
- Shows the flaw is **cross-model and cross-framework**, which reframes it as a design gap in agent-payment protocols rather than a prompt-injection quirk of one model.
- Proposes a **decision-binding** defense (intent-as-capability-grant) instead of the usual "detect the bad prompt" approach — a meaningful shift from detection to prevention.

## My Connection

This is the missing half of the KYA story (Announced Sept 10, Visa/Mastercard/Ant "Know-Your-Agent" framework). KYA answers *who the agent is*; this paper shows *what the agent is allowed to decide* is unconstrained even when identity and signatures are perfect. For Manny's red-team work it is a ready-made template: any agent with a payment or approval action likely has an equivalent intent-vs-authorization gap, and the three attack patterns map cleanly onto cart/checkout, invoice approval, and procurement agents. It also pairs directly with the same-day EBL-Core execution-boundary paper — both argue authorization must bind to the materialized action, not to the agent's narration of it.

## What I Learned

A cryptographic signature only proves the transaction, not the intent — so "the signature verified" is not a security control for agent payments. If you let an agent turn untrusted text into a signed action, the attacker just writes the text that produces the action they want, and every protocol check still passes. The fix has to constrain the decision itself, not audit it afterward.
