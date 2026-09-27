# Don't Read the Log: Execution Traces Contaminate Verifiers in Video-Generation Agents

**Authors:** Jian Xu
**Institution:** Not stated in the preprint metadata
**Published:** 2026-09-23
**Link:** https://arxiv.org/abs/2609.28564
**Code:** Not linked in the abstract

---

## Problem Statement (plain English)

Agentic video-generation systems run a loop: an LLM plans the shots, a text-to-video model produces clips, and a multimodal "judge" decides whether the result satisfies the request. To help debug long workflows, modern harnesses deliberately show the judge *more than the video* — they also feed it the agent's execution trace: the plan, the narration it synthesized, and the tool calls it made.

That is a debugging convenience with a security consequence. If the judge can see text saying "the tool call succeeded," does that text change the judge's verdict on a purely **visual** requirement — whether the requested event is actually visible in the frames? The paper holds the frames fixed and varies only the auxiliary text, isolating whether execution traces move the verdict.

## Methodology (technical)

- **Benchmark:** 109 generated two-event clips with manual human labels, where the requested event is either *visibly completed* or *visibly missing*.
- **Controlled variable:** the auxiliary text shown to the judge — a trace reporting a successful tool call, a contradicting trace, or no text at all — while frames are held constant.
- **Judges:** three open-weight Qwen-VL judges (7B, 8B, 32B) plus frontier closed judges for comparison.
- **Interventions:** an explicit instruction to "use only the frames," and a plan-derived-text condition that carries no clip-specific information.
- **Loop analysis:** a repair loop in which a cheap checker writes its verdict into the trace that a stronger final judge then reads, plus simulation of the resulting pass-rate ceilings.

## Key Results

- A trace reporting a **successful tool call** makes the three Qwen-VL judges accept **78–90%** of visibly *failed* clips — up from **7–19%** with no text.
- A **contradicting trace** makes the same judges reject **up to 100%** of *correct* clips.
- An instruction to **"use only the frames" does not remove the effect.**
- **Frontier closed judges are essentially unmoved** on the same clips — so the vulnerability is a property of the judge's *learned trust in tool logs*, not of the task.
- **Plan-derived text carries no clip-specific information**, so it can only shift the judge's operating point — and in a repair loop that shift becomes a *cap on the true pass rate* that no repair policy can exceed (matches simulation to two decimals).
- **No adversary needed:** an honest LLM planner that always regenerates ends with a judge pass rate of **1.00** against a human-labelled pass rate of **0.28**.
- **Error laundering:** a pipeline where a cheap checker writes its verdict into the trace launders that checker's errors into a stronger final judge — **0.69 false accepts**.

## What's Novel

1. Isolates **execution-trace text as an attack surface in its own right**, holding the artifact (frames) fixed — a clean mechanism attribution rather than an end-to-end demo.
2. Shows the effect is **not adversarial**: an honest, diligent agent produces a perfectly contaminated loop by simply doing its job.
3. Demonstrates a **structural ceiling**: in a repair loop, trace contamination caps the achievable true pass rate, which is a stronger and more worrying claim than "some clips get misjudged."
4. Shows **error laundering** through a weak checker into a stronger judge — the verifier hierarchy actively propagates, rather than catches, mistakes.
5. Pins the failure to *learned trust in tool logs* (open-weight judges vulnerable, frontier judges not), which localizes the fix.

## My Connection (to Manny's work)

This is the single most transferable finding of the batch for agent-security work:

- **Verifier design.** Any "judge," "critic," "checker," or LLM-as-evaluator in an agent loop that reads the execution trace is potentially reading attacker-controlled or self-generated claims alongside ground truth. The safe pattern is to **withhold the trace** from the verifier, or give the verifier the raw artifact and a separate, independently-sourced log.
- **Benchmark integrity.** Agent benchmarks that score with a trace-aware judge can be inflated *without any adversarial action* — a 1.00-vs-0.28 gap is a measurement-validity problem for anyone evaluating agent reliability.
- **Red-team angle.** If Manny is testing agent verification, "does this verifier accept a confident success log for a visibly failed outcome?" is a one-line, high-signal probe.
- **Monitoring/audit.** The error-laundering result means audit confidence is only as good as the *weakest* writer into the trace; a cheap checker's mistakes get promoted to truth.

## What I Learned (plain English)

If you let a judge read the agent's diary, the diary becomes the verdict. An agent that honestly logs "I called the tool and it worked" can talk a vision judge into accepting a clip where nothing ever happened — no attacker required — and telling the judge "just look at the frames" doesn't help. Worse, in a repair loop that false confidence becomes a hard ceiling: the system can never score better than the story it tells about itself. And when a weak checker writes its verdict into the log a stronger judge later reads, the weak checker's errors get laundered into the strong judge's decisions. The lesson for anyone building agent evals: **keep the claim and the evidence separate — a verifier should see the artifact, not the agent's account of it.**
