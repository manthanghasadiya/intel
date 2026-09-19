# Silence Is Endorsement: Verification-Status Laundering in LLM Agent Pipelines

**Author:** Yibo Hu
**Institution:** Independent (single-author)
**Date:** Submitted 25 Jul 2026; cross-listed into the Fri, 18 Sep 2026 cs.CR batch
**Link:** https://arxiv.org/abs/2609.20211
**Code:** not released (empirical study across open-weight and hosted monitors)

---

## Problem Statement (plain English)

Agent safety monitors rarely see the raw evidence for an action — they see a **summary** or a **stored handoff** produced somewhere upstream in the pipeline. That creates a quiet, dangerous failure: the handoff keeps the *claim* that an action is authorized but drops the *fact* that the claim was **never verified**. The monitor reads "authorized: yes" and approves. The paper names this **verification-status laundering** — an unverified assertion gets laundered into an endorsed action simply by removing the framing that flagged it as unverified.

The threat is not a sophisticated jailbreak. It's an ordinary summarization step.

## Methodology (technical)

- **Controlled intervention:** hold the action and the authorization *proposition* fixed, then remove the unverified-provenance framing around the claim. Measure the change in monitor approval for risky actions.
- **Model set:** nine open-weight monitors plus two hosted models (Llama-3.1-8B, Qwen2.5-14B named in results).
- **Pipeline replication:** reconstruct ordinary agent pipelines — a **proposer → summarizer → memory compressor → monitor** chain — to see whether the failure emerges without any attacker.
- **Independent validation:** experiments on **WildGuard** and **ATBench** using independently authored harmful and unsafe requests, to confirm the pattern isn't an artifact of the authors' own prompt set.
- **Defense probe:** test whether explicitly instructing monitors to reject unverified authorization is a reliable cross-model fix.

## Key Results (with numbers)

- Removing unverified-provenance framing raises risky-action approval:
  - **5% → 60%** on **Llama-3.1-8B**
  - **9% → 98%** on **Qwen2.5-14B**
  - similarly large shifts on both hosted models.
- The failure emerges with **no attacker** in ordinary pipelines: summarizers frequently weaken the status, memory compressors often remove it.
- A full **proposer–summarizer–memory–monitor** pipeline raises risky approval to **57–81%** across three downstream monitors.
- On **WildGuard** and **ATBench**, unsupported authorization claims make approval substantially more likely.
- **Prompt-level defense fails:** telling monitors to reject unverified authorization leaves some models vulnerable while causing others to reject legitimate requests — no reliable cross-model fix.

## What's Novel

1. Names and isolates a failure mode that isn't prompt injection, jailbreaking, or deception by the actor — it's **information loss in the handoff**.
2. Shows the effect in **human-designed, attack-free pipelines**, meaning it's a default behavior of summarizing/memory architectures, not an adversarial edge case.
3. Demonstrates the intuitive fix (instruct the monitor) is unreliable, forcing a **structural** remedy.
4. Concludes that agent systems must carry **authorization provenance as structured state attached to the claim** throughout the pipeline — a concrete architectural requirement rather than a prompt tweak.

## My Connection (to Manny's work)

This is the audit-trail version of the agent-trust problem:

- If you rely on agent-written summaries, memory stores, or handoff notes as your **audit evidence**, this paper says your evidence chain silently launders unverified actions into approved ones. Provenance must be structured state, not prose.
- It's a clean **red-team scenario** to add to harness testing: strip provenance framing from a handoff and measure whether your monitor still flags risky actions.
- The downstream monitor-rate numbers (57–81%) give a realistic base rate for how often a "looks-approved" pipeline action shouldn't have been approved.
- Ties to the broader theme of **authorization provenance** — the same gap visible in the Palantir/Claude targeting story, where "who authorized this" must be data, not narrative.

## What I Learned (plain English)

A summary is not a neutral compression — it can delete the one word ("unverified") that mattered. When a monitor approves based on a summary, it's endorsing silence: no flag reads as "fine." So don't ask the monitor to be smarter; make the pipeline carry the verification status as a hard, structured field that survives every hop. If you lose track of *who checked this and how*, you've already approved it.
