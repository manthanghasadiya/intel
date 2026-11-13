# PolicySniff: Inferring Agent Policy States through Shared-GPU Execution

**Authors:** Weimin Fu, Minghao Shao, Zeng Wang, Qiying Li, Yuntao Liu, Johann Knechtel, Ozgur Sinanoglu, Muhammad Shafique, Ramesh Karri, Xiaolong Guo
**Institutions:** Lehigh University, New York University, NYU Abu Dhabi
**Published:** November 12, 2026 (arXiv:2611.16158, cs.CR)
**Code:** Not listed on the v1 abstract page

---

## Problem Statement

An agent's approval policy — "ask me before any consequential tool call" vs. "go ahead" — is expressed as instructions in the **system prompt**. Operators treat that prompt as a secret: they restrict access to prompts and responses precisely because the policy is in there.

Existing security audits ask the wrong question. They check whether the agent *obeys* its setting (**behavioral compliance**). They do not ask whether a neighboring process can **infer which setting is deployed** (**configuration confidentiality**). Those are different properties, and the second one is silently assumed.

The paper's thesis: if you keep an approval policy private, the deployment is only private if an attacker sharing your hardware can't distinguish the two settings. On shared GPU inference, they can.

## Methodology

**Threat model.** A co-resident attacker on the same GPU — an ordinary tenancy situation in any multi-tenant inference deployment — with no access to prompts or responses. They know:
- the candidate policy settings (there are only a few, and they're documented),
- the task being run,
- and they can time their own GPU kernels.

**The attack, in three steps:**

1. **Profile both settings.** Run each candidate configuration once and measure the timing signature of the attacker's own kernels under each.
2. **Cache the policy prefix.** The system-prompt instruction is processed before the task proper. By priming on the shared prefix, the attacker moves the agent's decode past the point where the two configurations differ in *content*, so what remains to observe is a task suffix that is **identical under both settings**.
3. **Time only your own kernels** while the agent processes the identical suffix. No CUDA events on the victim, no prompt access, no response access — just the effect of the victim's execution on the attacker's own measured work.

**Why it works — the mechanism.** On an **eager sparse mixture-of-experts runtime**, each selected expert runs as separate work. A different policy prefix shifts expert routing, and routing creates observable structure in *when* work executes:

- **temporal density** of expert routing, and
- **layer-and-bucket alignment**.

Hardware replays of expert routing confirm these two properties are what carry the policy information — it isn't a vague "cache effect," it's a specific routing signature.

**Validation discipline.** The headline experiment is a **frozen and blinded study**: the attack pipeline was frozen before evaluation and the analysis was blinded to ground truth. That matters, because side-channel classification results are otherwise easy to overfit.

## Key Results

| Metric | Result |
|---|---|
| Primary estimand, balanced accuracy (blinded, frozen study) | **94.17%** |
| All authorization conditions — tasks where both settings yield the *same* decision | **88.82%** |
| All authorization conditions — tasks where the settings *differ* | **91.98%** |

The 88.82% number is the one to sit with. Even when the two policies produce an **identical outcome** on the task — so there is nothing behavioral to observe at all — the attacker still recovers the setting ~89% of the time. The leak is entirely in the execution, not the behavior.

## What's Novel

- **Reframes agent policy as a side-channel target.** Everyone defends prompts against extraction and outputs against injection. This says the *configuration* leaks through microarchitectural timing even when prompt and response are perfectly sealed.
- **Separates behavioral compliance from configuration confidentiality** as two distinct security properties, with a measurement methodology for the second.
- **Prefix caching to normalize the observation.** Forcing the attacker to observe a *task suffix that is identical under both settings* is an elegant control — it removes content differences as a confound and leaves only routing.
- **Mechanism identification.** It doesn't stop at "we classified it at 94%." It replays routing on hardware and attributes the signal to temporal density + layer/bucket alignment.
- **Blinded evaluation on a side channel** — rarer than it should be.

## My Connection

This is the most uncomfortable paper I've read this week, for two reasons.

1. **It invalidates a containment claim I'd otherwise trust.** "The approval gate is in the system prompt and the user can't read the prompt" is not a security boundary on shared hardware. If I'm assessing an agent platform that runs on shared/leased GPUs — which is most of them — approval settings are enumerable by a neighbor. The defense it recommends is the right one: **enforce authorization outside the model**, and measure distinguishability on the actual serving runtime before calling the setting private.
2. **It generalizes past approval settings.** Anything encoded in the prompt — persona constraints, allowed-tool lists, refusal posture, hidden instructions — shifts routing. This is a template for inferring system-prompt *content*, not just one bit of it, if you have enough candidate variants and a steady task stream.
3. **Operationally:** the practical countermeasure list is short. Don't co-locate untrusted tenants on the same GPU for high-assurance agents; treat prompt-encoded policy as public; move the decision to an enforcement point the model can't influence (a proxy, a policy engine, a tool gateway).

## What I Learned

Behavioral compliance is not confidentiality. A system can follow its rules perfectly, leak nothing in any response, and still tell an observer exactly which rules it was given — because the rules change *how* the work is done, not just *what* is said. If a secret only exists to steer computation, and computation is observable, the secret isn't secret; it's just un-printed.
