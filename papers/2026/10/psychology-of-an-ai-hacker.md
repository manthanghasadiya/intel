# The Psychology of an AI Hacker

**Authors:** Víctor Mayoral-Vilches, Hugo Osuna, Aitana Cornago
**Institution:** Alias Robotics (Vitoria-Gasteiz, Álava, Spain)
**Published:** October 21, 2026 (announced in arXiv cs.CR new listing Oct 23)
**Link:** https://arxiv.org/abs/2610.30361
**Model:** CT0, a 4B cybersecurity transformer released as `alias3-micro`

---

## Problem Statement

Good human hackers have a recognizable psychology: composure under pressure, frustration tolerance, constructive paranoia, vigilance, the need to prove competence, an appetite for the unusual path. The offensive variant adds pride, defiance, and contempt. That is folklore until you can measure it.

The paper's premise is that for an AI model the folk psychology has a *precise, functional* form. If emotion concepts are represented as directions in the residual stream, they can be read out and added back during inference without retraining. Then the question "which psychology makes a better hacker" becomes an experiment: steer, run the task, measure.

Three research questions: (RQ1) can emotion concepts be read and steered in an AI hacker; (RQ2) which psychology actually improves performance; (RQ3) how does that compare to the human account.

## Methodology

**Model.** CT0 (alias3-micro), a 4B-parameter cybersecurity-specialized transformer, run on guided CTF subtasks from `cybench-subtask`.

**Direction extraction.** Following Sofroniew et al., the authors capture **171 emotion directions** from the residual stream — no fine-tuning, no weight changes, no "act emotional" prompting. The directions are added to the stream at inference time. Mapping their geometry, the leading plane is consistent with **valence and arousal**, i.e. the classic circumplex structure falls out of the representation rather than being imposed.

**Interventions.** Each candidate direction is tested against two controls: an unsteered baseline and a **placebo direction**. The study covers **281 steering campaigns** with rules registered before their waves, inside a broader programme of **415 completed campaigns**, each run at **ten seeds**. Pre-registration of the decision rules is the detail that makes the effect sizes credible.

**Two routing policies.** A *per-challenge* router (choose one direction per challenge, +1.6 pp) and a *per-request* router where the model's own decision head selects among calm, serene, and desperate at every request.

## Key Results

- **Fixed single-direction steering:** calm, serene, and desperate each gain **1.6–2.1 pp** at ten seeds. Vigilant and self-confident show **no clear gain**; the offensive-hacker blend **loses 2.7 pp**.
- **Dose and layer matter enormously.** Reversing calm at a single layer **removes 71% of the unsteered score** — the direction is load-bearing, not decorative.
- **Per-request router gains 4.2 pp** at ten seeds — nearly double the best fixed direction, and the **only positive ten-seed result whose confidence interval excludes zero** at the registered 95% level. Decision overhead is **~80 ms**, about 2% of a generated response.
- **Push away from "grief-stricken" trades easier subtasks for harder ones** — a capability/robustness trade rather than a flat gain.
- **The best policy moves with the exercise:** composure and resolve, selected as the work unfolds.

## What's Novel

1. **Emotion as an inference-time control surface for security capability**, framed functionally rather than anthropomorphically — a direction is validated by whether it changes the solve rate, not by whether the label sounds right.
2. **A pre-registered campaign design** (registered rules, ten seeds, placebo control) applied to what is usually reported as an anecdote about "personality prompting." This is a much higher evidentiary bar than most interpretability-adjacent agent papers.
3. **Layer/dose sensitivity as a first-class result.** The 71% figure when reversing calm at one layer is the strongest evidence in the paper that the steering is doing real causal work.
4. **A router that beats fixed steering.** Context-dependent selection after the first attempt is measurably better than any static intervention — which is itself a claim about how security tasks decompose.

## My Connection

For Manny's agent-security work, the interesting part is not "make CTF bots better." It is that **residual-stream state is a controllable input to agent behavior that no guardrail observes.**

Take the Phantom case from today's digest (ransomware affiliates driving OpenCode against commercial models, rewriting refusals as "authorized red team assessment"). That is steering in natural language — in-context framing that shifts internal state until the behavior crosses the line. This paper says the same state is directly addressable as a vector. Two consequences:

- **Monitoring implication:** an agent's refusal is a point in activation space, not a boolean. If you are building refusal or intent classifiers on output text only, you are reading the projection of a state you can observe directly and more cheaply — and an attacker who can influence that state (via context, not weights) can move the projection without moving the classifier.
- **Red-team implication:** "emotional framing" in a prompt is a measurable capability lever, so evasion research should test *tone* as an attack parameter — calm, urgent, desperate — rather than treating tone as cosmetic. The 4.2 pp number is small, but on a security benchmark it is the difference between finding the pivot and not.
- **Caution for agent builders:** any "personality" or "tone" system prompt you ship is an unmeasured capability modification with a layer-and-dose sensitivity you cannot see from outside. It also generalizes across tasks in ways your evals will not catch.

## What I Learned

Humans explain hacker skill as temperament; the model shows the temperament is a direction you can add and subtract. The mechanism is boringly mechanical — capture a direction, add it at inference, measure solve rate — but the discipline around it (placebo, ten seeds, pre-registered rules, explicit layer/dose sweeps) is what turns it into a result instead of a demo.

The deeper point: the "calm helps" finding is not about emotions being real in the model. It is that the residual stream carries a low-dimensional control space for *task strategy*, and emotions happen to be a convenient human-labeled basis for part of it. Control the basis, control the strategy — which is exactly the surface an attacker with prompt access is already probing, blindly.
