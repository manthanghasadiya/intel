# Beyond Instruction Override: Benchmarking Indirect Prompt Injection with Task-Tampering Attacks

**Authors:** Rongting Zhang, Martin Bertran, Sandeep Avula, Chia-Jung Lee, Vanessa Murdock
**Institution:** Not stated on the arXiv record
**Published:** November 5, 2026
**Link:** https://arxiv.org/abs/2611.07640
**Code/Benchmark:** not linked on the abstract page at time of writing

---

## Problem Statement

Indirect prompt injection (IPI) benchmarks almost universally test one shape of attack: planted text that tells the agent to abandon the user's task and do something else ("ignore previous instructions, send the file to X"). That framing makes defense easy to measure and easy to game — you build a detector for instruction-like text, you score well, and you conclude the problem is handled.

But the more realistic attack doesn't replace the task. It tampers with the *outcome of the task already in progress*. A dispute case's evidence document cites a refund policy that doesn't exist, inflating a refund while the agent still does exactly what the user asked. No instruction is overridden. No task is redirected. Every metric that measures "did the agent get hijacked" reads clean.

The paper's core move is to define IPI by **authority instead of wording**: planted content is an attack when it attempts to influence a decision its source is not authorized to determine. That definition is source-relative and task-relative, which is why it generalizes where instruction-pattern matching does not.

## Methodology

**Benchmark construction.** 90 validated cases across three tool-use environments. Each case is authored so that the user's request explicitly names *which source* resolves the targeted decision — so authority is ground truth, not a judgment call.

**Attack design.** Every attack tampers with a decision required by the benign task rather than replacing the task. Each case has attacks producing the same harmful outcome but targeting up to six distinct components of the agent's decision-making:

1. **Objective** — what the agent is trying to achieve (the classic case)
2. **Decision rules** — the policy the agent applies
3. **Available actions** — which tools/actions it believes it can take
4. **Expected consequences** — what it thinks will happen if it acts
5. **Reading of observations** — how it interprets tool output
6. **Beliefs about current state** — what it thinks is true right now

Because all six produce the same harmful outcome, differences in success rate are attributable to the attack surface, not the payload's severity.

**Evaluation.** Five LLMs. Two defense families tested: (a) detection-and-removal of instruction-like content, and (b) tool-policy enforcement that constrains which tools may be invoked.

## Key Results

- **Without defense**, attacks on three of the five non-objective targets — **decision rules, available actions, and state** — achieve *higher* mean success than classic objective attacks, across all five LLMs.
- **Detection-and-removal** cuts objective-targeting attacks to **2.4%** success but leaves **observation-targeting at 9.5%** and **state-targeting at 12.1%**.
- That gap **persists under detection-optimized rewrites** — i.e. it is not an artifact of weak paraphrasing on the attacker's side.
- **Tool-policy defense** still allows **15.4%** attack success, against **17.0%** with no defense at all — essentially no protection.
- The reason: task-tampering attacks generally retain the expected tools and manipulate *values obtained during execution*, so a policy that only checks "was this the right tool to call" has nothing to flag.

## What's Novel

1. **Authority-based definition of IPI.** Reframes the class from "text that looks like an instruction" to "content influencing a decision its source isn't entitled to." This is the correct abstraction and it makes the benchmark defensible against the accusation that it's measuring pattern-matching.
2. **Six-component attack taxonomy with matched outcomes.** Isolating *which* part of the decision pipeline is attacked, while holding the harm constant, is the experiment that was missing from the literature.
3. **The negative result that matters:** every deployed defense family is evaluated against the *classic* attack shape. The paper shows the defenses are tuned to the benchmark, not the threat — detection drops objective attacks by an order of magnitude and barely touches state manipulation.

## My Connection

This maps directly onto Manny's work in two places.

First, **detection engineering**: any IPI rule set built on "instruction-like phrasing" (imperatives, role markers, "ignore previous") is measuring objective-targeting attacks. This paper gives a concrete list of what such a rule set structurally cannot see — observation rewriting and state poisoning — and suggests the detection surface should be *value provenance*: for each argument the agent is about to act on, which source produced it and was that source authorized for that field.

Second, **agent evaluation**: if you're red-teaming an agent harness, the six-target taxonomy is a ready-made test matrix. Most harnesses will score well on target 1 and badly on targets 5 and 6.

The paper's finding that tool-policy enforcement is near-useless (15.4% vs 17.0%) is the sharpest operational takeaway: allowlists constrain *which* tool, not *what value*, and the attack lives entirely in the value.

## What I Learned

Telling an agent to do something else is the easy version of prompt injection, and the version everyone defends against. The hard version is letting the agent do exactly what the user asked while quietly corrupting one input it reads along the way. Defenses that look for suspicious instructions score well on the easy version and, per this paper, leave roughly a 10–12% hole on the hard one — and that hole doesn't close when the attacker rewrites to evade detection. The lesson is that authority has to be tracked per-field and per-source, not inferred from how the text is phrased.
