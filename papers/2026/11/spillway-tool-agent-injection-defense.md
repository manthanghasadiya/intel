# SPILLWAY: Separating Target Admission from Sanitization to Defend Tool-Using Agents against Prompt Injection

**Authors:** Woosang Lim (SAINT Machines / KAIST), Yeseul Chang (Chung-Ang University), Jaewoo Lee (Chung-Ang University)
**Institution:** SAINT Machines, KAIST, Chung-Ang University
**Published:** Submitted November 4, 2026 (arXiv:2611.07222)
**Link:** https://arxiv.org/abs/2611.07222
**Code:** Project page + per-cell results linked from the abstract page
**Venue:** Earlier version accepted at the Third Workshop on Agents in the Wild: Safety, Security, and Beyond (NeurIPS 2026)

---

## Problem Statement

Tool-using agents act on email, files, and payments. An instruction hidden in a web page or a shared document can therefore make an agent send data — or money — to an attacker's target. The industry has responded with runtime defenses, and nearly all of them share one structural weakness: **they put a language model on the decision that lets an action through.** That model has to read the attacker's text to decide whether the attacker's text is dangerous, and it costs a model call at every single step.

The alternative — provenance defenses — keeps the model off the final check but requires a model to plan or route each task up front. So you either pay per step, or you pay for a plan that itself can be poisoned.

The authors ask whether the *where* of an action can be decided without a model at all.

## Methodology

The core insight is a type distinction that already exists in the tool interface:

- An **email's sender** and a **payment's account** arrive in a **typed field** of a tool's return value.
- Injected instructions arrive as **free text**.

SPILLWAY builds two deterministic components on that distinction.

**1. Target admission (the floor).** A tool call is admitted only when every target value that appears in tool output but *not* in the user's task was either:
- returned by a read-only tool in a **typed field**, or
- covered by a **fixed delegation rule**.

Anything else is refused. Notably: there is **no per-task plan**. The check is a static policy over typed returns, which is why it costs no backbone re-run and almost no added model call.

**2. Deterministic sanitizer (pre-model).** Instruction-like regions matched by fixed rules are removed *before the model reads them*. The removed spans are retained as evidence — so the sanitizer is auditable rather than a silent filter.

**3. Local judge (bounded restoration).** A local judge may **restore a refused call** — but never one whose target lies inside the removed text. That asymmetry is the safety property: the judge can fix over-refusal, it cannot be talked into re-admitting content the sanitizer caught.

**Evaluation:** three backbone models, and seven task suites on one of them. Attack conditions include standard templated injection, a forged chat template, and a **released reinforcement-learning attacker**.

## Key Results

- **Zero** standard templated injections redirect an action to an attacker-controlled target under SPILLWAY — across all three backbones.
- Highest mean utility of any defense run under that attack, plus the best composite scores (NRP, ERS).
- A **forged chat template** attack produces a raw zero.
- Mean **in-scope attack success under a released RL attacker: 0.007**.
- Achieved with **no defense-triggered backbone re-run** and almost no additional model calls.

The utility-parity result matters as much as the ASR result: prior defenses that stop injections usually buy that with refusals on legitimate work. SPILLWAY's floor is set by typed returns, which legitimate work also uses.

## What's Novel

1. **Reframing injection defense as a type problem, not a content problem.** The target of an action is structurally distinguishable from instructions about it. That is a data-flow property, not a classifier judgment.
2. **Removing the per-task planner.** Prior provenance work needed a model to plan routes. SPILLWAY's admission rule is static policy over typed returns — no plan to poison.
3. **Sanitization as evidence, not just filtering.** Retaining removed spans makes the defense auditable and produces a forensic record of attempted injection.
4. **The asymmetric judge.** Restoring refusals only where the target is *not* in removed text gives you over-refusal recovery without a new bypass surface.

## My Connection

This is directly load-bearing for any agent Manny builds or attacks that touches mail or money. Three specific uses:

- **As a design pattern**: if you're writing an agent that pays invoices, admits files, or sends mail, the typed-return admission rule is implementable in a day and does not require a model in the hot path.
- **As an attack-planning counter**: if a target claims SPILLWAY-style provenance defense, the interesting questions become (a) can you get the agent to treat attacker-influenced data as a typed field — e.g. spoof a `From:` header, or land a value in a "verified sender" column; (b) can your payload survive *inside* a typed field (a display name, a memo line, a payee note); (c) does the delegation rule have a wildcard. Every one of those is a "policy expressiveness" bug rather than an injection bug.
- **As a benchmark bar**: 0.007 mean ASR under an RL attacker is the number to beat when you evaluate a client's agent runtime. Ask vendors for their score under an adaptive attacker, not a templated one.

## What I Learned

Most prompt-injection defense is an argument about *text*. SPILLWAY is an argument about *types*: the instruction and the destination of an action arrive through different channels, and you can decide admission on the channel rather than on the content. That is why it is cheap and why it holds up — a deterministic check on where data came from cannot be socially engineered the way a judge model can. The generalizable lesson beyond this paper: when you find yourself putting an LLM in a security decision loop, look first for a structural invariant (a type, a signature, a provenance tag) that makes the decision unnecessary. The best agent security work this month — this, and the deterministic gates in the early-November MCP papers — is all the same move.
