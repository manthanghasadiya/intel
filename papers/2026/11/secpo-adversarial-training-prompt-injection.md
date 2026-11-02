# SecPO: Principled Adversarial Training for Prompt Injection Security

**Authors:** Sizhe Chen, Yibo Peng, Jaewon Chang, Chawin Sitawarin, David Wagner
**Affiliation:** UC Berkeley (co-authors also affiliated with Anthropic and Google DeepMind — work done at GDM)
**Published:** October 30, 2026 · **Link:** https://arxiv.org/abs/2610.41998
**Code:** https://github.com/Sizhe-Chen/SecPO · **Data:** Qwen3.6-27B-Instruct-SecPO-trainset · **Weights:** Sizhe-Chen/Qwen3.6-27B-Instruct-SecPO (all on Hugging Face)

---

## Problem Statement

The best publicly-defended LLM for prompt injection, Meta-SecAlign, is trained with DPO — and still fails at close to 100% attack success under *adaptive* prompt injection. Two things are wrong with it:

1. **DPO anchors to an insecure reference model.** The loss keeps the policy close to the base model, and the base model has no injection resistance. You are optimizing toward a target that is itself undefended, so the KL anchor actively pulls the model back toward compliance.
2. **It trains on weak simulated attacks.** A defense trained on weak injections learns the shape of weak injections, not the shape of the attack.

The paper's contribution is fixing both: a theoretically principled objective (SecPO) that anchors to something that *is* injection-invariant, plus an adversarial-training pipeline that generates genuinely strong attacks.

## Methodology

**The key observation.** An idealized reference model — injection-invariant and utility-preserving — would be the right anchor for preference optimization. The problem is that you can't normally obtain its output distribution. The paper shows it *is* accessible during training: for a training example, the injection-invariant behavior is the model's own output conditioned on the trusted input alone, without the untrusted content. That gives a tractable approximation of the oracle distribution, and from it a principled loss (SecPO) rather than a heuristic.

**Adversarial training pipeline.** Generate strong injected inputs with PAIR (best@240 — 240 attack attempts, keep the best), then train on them. The point is transfer: a model trained on genuinely strong attacks should defeat *unseen* adaptive attacks at test time, which is the setting that matters.

**Model.** Qwen3.6-27B-Instruct as the base. Evaluated on AgentDojo (agentic tasks), DTaP-Bench (real-world agentic tasks with strong injections), and six other benchmarks spanning agentic tool calling, instruction following, science and math — the utility half of the security/utility tradeoff.

**Threat model note worth flagging:** the paper defines prompt injection under the assumption that the agent is *told* which inputs are trusted and which are untrusted. It explicitly excludes cases where the trust label must be inferred from context. That's the standard formalization, and it's also the gap that real deployments (where an email's trustworthiness is exactly what's in question) fall through.

## Key Results

- **AT alone (vs Meta-SecAlign):** 1.5–14.5× reduction in ASR across three state-of-the-art adaptive attacks.
- **SecPO loss on top of AT (vs DPO loss):** a further 2.4–4.6× reduction.
- **AgentDojo ASR@800 under AttackerMovesSecond (the current strongest adaptive injection, 800 attempts per task):** **8.8%** for SecPO + AT vs **78.8%** for Meta-SecAlign. Same order of magnitude as a working defense rather than a sieve.
- **Max ASR across three SoTA adaptive attacks:** **13.3%** vs **94.0%** for Meta-SecAlign.
- **DTaP-Bench (real-world agentic tasks, strong injections):** **7.6%** ASR.
- **Utility:** no noticeable reduction across nine benchmarks covering agentic tool calling, instruction following, science and math.
- **Matches GPT-5.5's measured security** while being at least two orders of magnitude smaller and using a single defense scheme rather than multiple proprietary ones.
- **Released training set** (with the optimized injections baked in) yields 2.5–22.5× lower ASR for anyone who trains on it, with no attack-generation cost of their own.
- **Emergent transfer:** the defended model resists adaptive *visual* prompt injections even though that wasn't a training objective.

## What's Novel

1. The theoretical move: showing the injection-invariant oracle's output distribution is approximable from training data, which converts "DPO but with a better anchor" from a wish into a loss function. This is the part that makes it principled rather than another empirical tweak.
2. Combining a corrected objective with genuinely strong adversarial training — the two failures of Meta-SecAlign were independent, and fixing only one leaves the other.
3. Releasing the *attack-optimized training set*, which is arguably the most practically useful artifact here: adversarial training is expensive because generating best@240 attacks is expensive, and this removes that cost for everyone downstream.
4. Emergent robustness to visual injections, which suggests the defense is learning something about instruction/data separation rather than pattern-matching a specific attack family.

## My Connection

- **This is the defense side of every injection finding in the digest.** Attack success rates near 100% are usually reported against undefended or DPO-defended models; 8.8% at 800 attempts per task is a different bar, and red-team reports should state which bar they cleared.
- **Attack budget matters more than attack cleverness.** The evaluation is ASR@800 — 800 attempts per task. If your engagement only tries 5 payloads against a SecPO-class model and gets 0 hits, that's not evidence of robustness.
- **The trust-label assumption is the exploitable seam.** The defense presumes the agent knows which content is untrusted. Any scenario where that boundary is fuzzy — a forwarded email, a document from a colleague, a page whose provenance is unclear — sits outside the formal guarantee. That's the red-team entry point, and it's a legitimate, publishable critique.
- **Practical adoption:** the released training set means a small team can fine-tune injection resistance without an attack-generation cluster. Worth testing against an internal model.

## What I Learned

Prompt injection defense kept failing because the training objective was pointing at the wrong target. DPO says "stay close to the model you started from," and the model you started from will happily follow instructions in a PDF. SecPO says "stay close to what you would have done if the untrusted content weren't there" — which is precisely the behavior you want, and it turns out you can compute an approximation of that during training.

The second lesson is that defense quality is bounded by attack quality during training. Meta-SecAlign wasn't badly designed so much as under-attacked: trained on weak injections, it learned weak injections. 240 PAIR attempts per example is the difference between teaching a model to recognize a specific trick and teaching it to separate instructions from data.

And the honest caveat: 8.8% is excellent relative to 78.8%, but at 800 attempts it still means roughly 70 successful injections per 800 tries. For an agent with a shell, that's not a solved problem — it's a problem you now have to catch downstream.
