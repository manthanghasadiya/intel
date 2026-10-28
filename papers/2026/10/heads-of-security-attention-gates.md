# Heads of Security: Learned Attention Gates Improve Prompt Injection Robustness

**Authors:** Igor Shilov, Xiaoxue Yang, Alexander Panfilov, Evgenii Kortukov, Matthieu Meeus, Peter Romov, Wojciech Samek, Jonas Geiping, Yves-Alexandre de Montjoye | **Affiliation:** not listed on the arXiv record
**Published:** October 27, 2026 | **arXiv:** [2610.37657](https://arxiv.org/abs/2610.37657) (cs.LG, cs.CL, cs.CR)
**Code:** not stated on the record

---

## Problem Statement

Every deployed prompt-injection defense makes the same implicit trade. You can bolt separation on from outside — a filter, a classifier, a prompt wrapper, a second model that reviews the first — or you can fine-tune the target model to prefer instructions from the user over instructions found in data. Both have failure modes that have now been measured repeatedly.

The external filter is a separate model that shares the target's blind spots and can be attacked on its own. The fine-tuned defense ("align the model to ignore injected text") degrades general capability, and it degrades worst exactly where agents live: few-shot prompting, unusual formats, tool use. And it is still a *behavioral* property — the model learned to behave as if instructions and data are separate, rather than the separation existing in the computation.

This paper asks a sharper question: can the separation be built into the attention mechanism itself, cheaply enough that it costs almost nothing at inference time and doesn't need to relearn the whole model?

## Methodology

**The intervention.** A small set of attention heads get a learned gate that decides, per head, how much attention to redistribute between *trusted* tokens (system prompt, user turns) and *untrusted* tokens (tool output, retrieved documents, web content). The tokens are tagged by provenance at the boundary — the model doesn't have to infer trust from content. HoS learns the gate; everything else stays frozen or lightly tuned.

**Why per-head rather than per-layer or global.** Heads specialize. Prior interpretability work shows a minority of heads carry most of the "follow the instruction in the text" behavior. Gating at head granularity lets the defense suppress the specific circuits that relay instructions out of untrusted spans without touching the heads that do useful work — which is exactly why utility survives.

**Evaluation setup.** Two base models: Llama-3.3-70B-Instruct and GPT-OSS-20B. Three baselines: the undefended base model, SecAlign++ (state-of-the-art fine-tuning-based defense), and ASIDE (architectural instruction-data separation — the closest prior art).

**Attack suite.** Two named adaptive attacks rather than a static benchmark:
- **RL-Hammer** — black-box reinforcement-learning attack that searches for injections against the deployed model.
- **Claudini** — white-box attack, then strengthened against HoS specifically through autoresearch, i.e. an attacker that knows the defense and iterates on it.

Plus standard agentic prompt-injection benchmarks with static attacks, and standard multiple-choice utility benchmarks across prompting setups (zero-shot, few-shot).

## Key Results

| Metric | Base | SecAlign++ | HoS |
|---|---|---|---|
| RL-Hammer ASR, GPT-OSS-20B | 97% | 85% | **10%** |
| Claudini FLOPs to reach same ASR | 1x | 1x | **~3x** |
| Few-shot utility | preserved | degraded | **preserved** |
| ASIDE agentic utility | — | — | ASIDE only 10% |

- **Black-box adaptive attack:** 97% ASR against the undefended GPT-OSS-20B, 85% against SecAlign, **10% against HoS**. That is roughly a 9x reduction against an attacker that can query the model.
- **White-box adaptive attack:** Claudini, strengthened specifically against HoS through autoresearch, needs about **3x the FLOPs** to reach the same success rate it achieves on the base model or SecAlign. The attack doesn't fail — it gets expensive.
- **Utility:** HoS preserves base-model performance on multiple-choice benchmarks and across prompting setups, while SecAlign and ASIDE degrade in few-shot settings.
- **Agentic benchmarks (static attacks):** HoS and SecAlign perform similarly and both substantially improve the security-utility trade-off over the base model. ASIDE collapses on tool use — **10% utility** — which is the tell that architectural separation done naively breaks the agent loop.

The 10% ASR number is the headline. The 3x-FLOPs number is the more honest one: against an adaptive white-box attacker this is cost imposition, not immunity, and the paper says so.

## What's Novel

1. **Separation as a mechanism, not a behavior.** Prior architectural work (ASIDE) separates instruction and data streams; HoS instead learns *where in the network* the relay happens and gates it. That's a lighter touch — no stream duplication, no rewrite of the forward pass — and it's why tool-use utility survives where ASIDE's doesn't.
2. **Provenance-typed tokens.** Trust is a label attached at the boundary, not something the model infers from phrasing. This is the correct design: it makes the security property a function of the harness's labeling discipline rather than of the model's world knowledge.
3. **Adaptive-attack discipline.** Both attacks are adaptive, one is white-box, and the white-box one was *strengthened against the defense* by autoresearch before being reported. Most defense papers still report static-benchmark numbers and call it done.
4. **A utility result that matters.** Beating SecAlign on few-shot and beating ASIDE on tool use are the two ways prior defenses actually failed in deployment.

## My Connection

This is the first prompt-injection defense I've seen that is plausibly *shippable by a model vendor* rather than bolted on by an app developer — which changes the threat model for anyone building agents on top of MCP. If separation lives in the weights, then the tool-response trust boundary stops being something each harness reimplements badly in a system prompt, and starts being a property of the model you call.

Two caveats I'd want before trusting it in a red-team report. First, the trust label is supplied by the caller — if your harness mislabels a tool response as trusted (or a retrieved document inherits the user's trust because it came back through a user-initiated call), the gate has nothing to work with. HoS moves the security-critical decision from "does the model resist this text" to "did the harness tag this text correctly," which is a smaller and more auditable problem, but it is still a problem. Second, 3x FLOPs against a white-box attacker is a cost the attacker may simply pay; the defense's value depends on whether that cost buys the defender detection time, and the paper doesn't measure that.

The obvious extension to test: does HoS survive an attacker who can also manipulate *which* tokens get labeled trusted — i.e. a poisoned MCP server that returns content inside a field your harness renders as user-authored? That is the realistic MCP attack, and it sits upstream of everything this paper measures.

## What I Learned

The interesting move here is refusing the framing "make the model more robust." Robustness is a behavioral claim and behavioral claims get eaten by adaptive attacks. What HoS does instead is make the *routing* of instruction-following conditional on provenance — a structural property that an attacker has to defeat by finding heads the gate missed, and the paper shows the gate generalizes across an autoresearched attack.

The transferable lesson for anyone building agent defenses: when a defense degrades utility, the usual cause is that it's suppressing a capability instead of a pathway. Suppress the pathway — the specific heads, the specific socket path, the specific syscall — and you get to keep the capability. That's the same shape as Tailscale's bug from today's digest, inverted: their forwarding check failed because it validated a *name* and then operated on the *object*. Security properties that attach to labels rather than to the thing itself are the ones that lose.
