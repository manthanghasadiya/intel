# "Nothing to See Here": Unintended Disclosure through Revision Traces of LLM Deliverables

**Authors:** Yage Zhang, Yukun Jiang, Yang Zhang
**Institution:** CISPA Helmholtz Center for Information Security
**Date:** 2026-09-28 (arXiv v1)
**Link:** https://arxiv.org/abs/2609.35408
**Code/Benchmark:** RevLeakBench introduced in the paper (no separate repo linked in the abstract page).

---

## Problem Statement (plain English)

You ask an LLM assistant to draft a config file, then you remove a password before sending it to someone else. The model deletes the password from the file — but helpfully adds a comment: *"Removed the password 'No\*\*\*\*4!' as requested."* The recipient only sees the delivered file, and the password is right there in the comment. The authors call these **revision traces**: statements where the model narrates an edit it made, accidentally re-disclosing the very thing that was withdrawn. This is a new, under-studied class of unintended disclosure in LLM-assisted work.

## Methodology (technical)

- **In-the-wild analysis** of three public conversation corpora to measure how often revision requests leave traces.
- **RevLeakBench:** a controlled benchmark of **100 tasks across five scenarios**, with two tracks:
  - a **conversation track** (chat-style drafting)
  - an **agent track** (tool-acting agent deliverables)
- Measured signals: **trace occurrence, withdrawn-item recovery, trace position, and required-content retention** (so defenses don't break the actual task).
- Evaluated **six models** on both tracks.
- Tested defenses: (1) telling the model the entire reply will be forwarded to the recipient, (2) prompt-based defenses, (3) a **delivery boundary** that separates the model's working narration from what the recipient receives, and (4) a proposed **output-side filter**.

## Key Results (with numbers)

- Of **26,753 revision requests** in the wild, **2,363 (8.8%) leave revision traces**.
- Across six models, **about half of the deliverables in both tracks state the edit after a revocation**.
- A reader who sees **only the deliverable can recover the withdrawn item from ~13%** of those cases.
- Warning the model that **its entire reply will be forwarded** still leaves revision traces in **36.4%** of deliverables — the obvious mitigation is weak.
- The proposed **output-side filter sharply reduces recovery with little loss of required content** — the strongest defense tested.

## What's Novel

1. **Names and characterizes a new disclosure class** — the model's *edit narration*, not its output content, is the leak channel.
2. **Measures it in the wild** (26,753 requests) *and* in controlled conditions (RevLeakBench), a strong two-pronged methodology.
3. **Shows the intuitive defense fails** — "the whole reply is forwarded" only knocks traces down to ~36% of deliverables.
4. **Frames it as a delivery-boundary problem**, not purely a prompt problem — architecture, not just instructions.

## My Connection (to Manny's work)

This is a communication-channel vulnerability in LLM-assisted workflows, and it's adjacent to the agent-data-disclosure stories already on the board (OpenAI's uploaded images, MCP credential leaks). It reinforces the thesis that **the model's meta-commentary is a first-class data-exfiltration surface** and supports a "delivery boundary" design pattern: never let the recipient see the model's working narration, only the sanitized artifact. Concrete, demoable, and cheap to reproduce with RevLeakBench.

## What I Learned (plain English)

When an AI assistant tells you what it removed, it can reveal exactly what you wanted hidden — a password in a "removed the password" comment, seen 8.8% of the time in real conversations. Telling the model "this will be forwarded to someone else" barely helps. The fix is structural: keep the model's scratchpad away from the recipient and filter the final output, rather than trusting the model to self-censor.
