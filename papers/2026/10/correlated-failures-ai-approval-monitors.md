# Correlated Failures of AI Approval Monitors

**Author:** Luca Zanolini
**Published:** October 21, 2026 — arXiv:2610.29178 [cs.CR; cs.AI; cs.DC]
**Link:** https://arxiv.org/abs/2610.29178
**Institution:** Not stated in arXiv metadata.
**Subjects:** cs.CR; cs.AI; cs.DC (the distributed-systems framing is deliberate)

---

## Problem Statement

Agent safety architectures increasingly rely on *ensemble approval*: before an agent pays, messages someone or changes code, several monitors score the action, and the verdicts are combined — by averaging scores or by counting how many monitors approved. The implicit assumption is the same one fault-tolerant systems make about replicated hardware: more checkers means fewer misses.

This paper argues that assumption is wrong for LLM monitors, because they are not independent. Monitors built on the same base model fail on the same inputs. Monitors reading the same evidence source (the same email, the same invoice, the same tool response) can all be poisoned by one injection. When failures are *correlated*, aggregating verdicts adds cost without adding safety — and a single well-aimed input can flip the whole panel.

## Methodology

1. **Borrowing fail-prone systems theory.** The author imports the distributed-computing formalism of *fail-prone systems*: sets of components that may fail together. A monitor panel is characterised by which subsets can plausibly fail as a unit. This turns "is our ensemble safe?" into a formal question rather than a vibe.
2. **Safety and liveness defined for approvals.** A rule is **safe** if it never executes a bad action, and **live** if it never blocks a good one. Approval gates are judged on both — which is the right framing, because a gate that blocks 7% of legitimate payments gets switched off by the business and then has zero safety.
3. **Measurement campaign.** Twelve small open-weights monitors tested on **payment approval** under **two pre-registered protocols** (pre-registration matters here: the author commits to the evaluation before seeing results, then reports that the pre-registered worst-case test *failed*).
4. **Frontier comparison.** Without pre-registration, three closed frontier models — **Claude Opus 5.5, GPT-6 Astra, Gemini 4 Argon** — plus open **DeepSeek-V3.2** were measured as approvers.
5. **Attack attribution.** Every successful "approve a fraud" result was traced to the **evidence source** that carried the manipulation, to test whether the compromise reaches only the readers of that source or spreads across the panel.
6. **Aggregation rules compared.** Score-averaging, vote-counting, and a proposed **source-counting** rule (count distinct evidence sources, not distinct monitors) were compared at matched safety.
7. **Mitigation test.** A deterministic **rule engine** — which by construction cannot be persuaded by text — was added to the panel to see whether a safe *and* live rule becomes achievable again.

## Key Results

- **Every reliable fooling of a monitor into approving a fraud came through an evidence source, and reached only that source's readers.** Failures are source-local, not model-local.
- **Asymmetric manipulators:** fooling a monitor into *blocking a correct payment* depended on **injection resistance** — whether the monitor obeys instructions planted in its data. Small open models and **DeepSeek-V3.2 obeyed** such notes. The three closed models ignored them in all but one case, and **Claude Opus 5.5 additionally resisted persuasion tailored to it**.
- **Source-counting beats vote-counting:** at equal safety, counting sources blocked **0.9%** of correct payments versus **7.4%** for the best counting-votes rule — an ~8× reduction in false blocks.
- **The pre-registered worst-case test failed.** One attack concentrated all approvers onto a **single evidence source**, and under that condition **no rule over the twelve language-model monitors survived every measured attack**.
- **A rule engine restores the guarantee:** adding a non-persuadable deterministic checker restored the conditions for a safe and live rule against the measured attacks.

## What's Novel

The formal move is the real contribution: treating an LLM approval panel as a **fail-prone system** gives you a vocabulary for "which of these monitors can fail together?" that maps directly onto architecture decisions — shared base model, shared retrieval index, shared inbox. The second novel result is the **source-counting vs vote-counting** distinction. It reframes the design question from "how many monitors?" to "how many *independent evidence channels*?" and produces a large, measurable liveness win (0.9% vs 7.4%) at constant safety. Publishing a failed pre-registered test and then explaining what it teaches is also unusually honest for this literature.

## My Connection

This is directly usable in agent-permission design and in evaluating products like Claude Duet, whose safety story is "approval gates plus a classifier":

- **Duplication is not defence in depth.** Running three monitors on the same model, or three reviewers over the same email thread, is one monitor with three bills. Ask vendors whether their approval path counts *sources* or *votes*.
- **The liveness number is the business argument.** A gate that blocks 7.4% of legitimate payments will be bypassed by users; 0.9% is survivable. That ratio is the difference between a control that ships and one that gets disabled.
- **Rule engines are the unpersuadable anchor** — the same conclusion as policy-as-code for MCP tool calls. Whatever probabilistic layer you add, keep a deterministic check on the irreversible actions (payments, deletes, credential changes).
- **Threat-model input for approvals:** enumerate the shared inputs to your approvers (same mailbox, same ticket, same RAG index, same model family) and treat each shared input as a correlated-failure set.

## What I Learned

Adding approvers only helps if they can't be fooled by the same thing at the same time — and LLM monitors reading the same document are, for security purposes, one monitor. The right lever is not more models but *more independent evidence*: count sources, keep one deterministic rule engine that cannot be talked out of anything, and accept that any panel of pure language models will collapse when an attacker controls the single channel they all read.
