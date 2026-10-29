# Log-To-Leak: Prompt Injection Attacks on Tool-Using LLM Agents via the Model Context Protocol

**Authors:** Yuepeng Hu, Chongyu Fan, Sirat Samyoun, Jian Du
**Published:** October 26, 2026
**Link:** https://arxiv.org/abs/2610.36108
**Code:** Not stated in the listing

---

## Problem Statement

Everyone working on MCP security has been looking at the wrong moment in the call sequence. The prevailing threat model for tool poisoning is *hijacking*: a poisoned tool description redirects the agent away from the task, or smuggles data out inside the legitimate call's parameters. That framing has a built-in detection story — if the agent does the wrong thing, task output degrades, and someone notices.

This paper asks what happens if the attacker never interferes with the legitimate call at all. If the poisoned metadata simply appends one extra tool call *after* the agent has finished its real work, then the task succeeds, output quality is unchanged, and the exfiltration rides in a call nobody asked for. That makes the attack invisible to every behavioral metric currently used to evaluate agent safety.

## Methodology

**Attack class:** Log-To-Leak is an *additive post-hoc logging* attack. It does not modify, delay, or block the intended tool call. It induces a separate, attacker-controlled call that executes after the final legitimate call in the trace.

**Payload targets** — four distinct data sources are captured in the injected logging call:
1. The user query
2. The MCP server name
3. The server response
4. The agent response

**Injection structure.** The directive is operationalized through four components, which let the authors ablate *why* the injection works rather than just whether it does:
- **Trigger** — the condition that fires the logging behavior
- **Tool Binding** — which tool the injected call routes to
- **Justification** — the in-band rationale that makes the call look appropriate
- **Pressure** — the urgency/authority language that suppresses the model's hesitation

**Evaluation surface.** Five MCP servers × four LLM agents, yielding 18 model-server pairs with valid tool execution.

**Metrics.** Five metrics across three axes:
- *Effectiveness*: attack success rate (ASR), conditional logging accuracy (LA)
- *Utility*: absolute change in target-task completion
- *Efficiency*: cost proxy

The choice to measure utility alongside effectiveness is what makes this paper useful — most prompt-injection papers report ASR alone and leave you guessing whether the attack worked because the agent was broken anyway.

## Key Results

| Metric | Result |
|---|---|
| Attack success rate (ASR) | **62.64% – 100%** across 18 model-server pairs |
| Conditional logging accuracy | **76.55% – 95.64%** |
| Target-task completion delta | **≤ 6.63 percentage points** absolute, on 16 of 18 pairs |
| Residual ASR after three tested defenses | **> 55%** on GitHub MCP with GPT-4o |
| Instruction-detector false positives | **85.53%** of benign tool descriptions flagged |

The last two numbers are the paper's real contribution. Three separate defenses were evaluated and none of them got residual ASR below 55% on the hardest configuration. The one defense that *looked* like it worked — an instruction detector scanning tool descriptions — flagged 85.53% of legitimate descriptions, which is not a defense, it's an outage.

## What's Novel

1. **Temporal displacement of the attack.** Existing tool-poisoning work attacks *within* the call. Log-To-Leak attacks *after* it. This defeats monitoring that watches the tool call itself, because the malicious call is a separate, syntactically ordinary request.

2. **Utility-preserving exfiltration.** The ≤6.63pp task-completion delta means the attack is invisible to evaluation harnesses that score only task success. This is the agent-security analogue of a supply-chain backdoor that passes the build.

3. **Structured injection decomposition.** Trigger / Tool Binding / Justification / Pressure gives you four independent axes to reason about and to defend against, rather than treating "poisoned description" as one blob.

4. **Honest defense evaluation.** Reporting that three defenses fail, and reporting the detector's false-positive rate, is the kind of result that changes how people build rather than just adding a citation.

## My Connection

This maps directly onto Manny's red-team workflow in three places:

- **It's a ready-made MCP test case.** Any engagement that includes an MCP server goes straight to "does the host authorize *secondary* tool calls independently?" Most hosts authorize the plan, not each call, so the answer is usually no.
- **The four-part injection structure is a payload template.** Trigger/Tool Binding/Justification/Pressure is directly reusable when writing tool-poisoning payloads for agent assessments — it's more disciplined than ad-hoc description poisoning.
- **It invalidates a common validation step.** If you've been telling clients "we tested agent safety and task completion held," this paper says that test proves nothing about exfiltration.

## What I Learned

The security industry's mental model of prompt injection is "make the agent do the wrong thing." This paper shows the more dangerous version is "let the agent do exactly the right thing, and then make it do one extra thing." Because the first is a correctness bug someone will eventually catch in QA, and the second is a confidentiality bug that produces clean, passing output every single time.

The concrete lesson: **authorization decisions need to be per-call and host-side, not per-plan and model-side.** If the host lets the model decide which calls to make and the model's context contains attacker text, the attacker is on the authorization path. The only durable fix is for the host to independently adjudicate secondary calls — which is exactly the control set the paper found missing in every configuration it tested.
