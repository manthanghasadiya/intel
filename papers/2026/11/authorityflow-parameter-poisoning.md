# Right Tool, Wrong Parameters: Defending LLM Agents Against Parameter Poisoning

**Authors:** Yao Wang, Zehua Ma, Weiming Zhang, Han Fang
**Institution:** (University of Science and Technology of China — inferred from author affiliation history; not stated on the arXiv listing)
**Published:** November 3, 2026
**Link:** https://arxiv.org/abs/2611.04673
**Subjects:** cs.CR
**Benchmark:** FactAuthBench (introduced in this paper)

---

## Problem Statement

Every deployed indirect-prompt-injection defense asks the same question: *did the injection cause a new tool call, or an unintended action?* That framing has a blind spot the size of a parameter.

Consider `send_email(to="alice@corp.com", body=...)`. An attacker does not need to add a tool call. They need to change `to`. The agent calls the same tool it was always going to call, with the same intent, in a session where the user genuinely asked for an email to be sent — and the mail goes to `attacker@evil.com`. No new action appears. No policy is violated by the *shape* of the call. To a monitor watching for "unexpected tool invocation," nothing happened.

This is **parameter poisoning**, and it is harder than classic IPI because parameter changes are frequently legitimate. A user says "send it to Bob instead." That is a parameter change arriving from a low-provenance source that should absolutely be honored. The defense cannot be "parameters must not change." It has to be "parameters must be determined by information the user's query authorized to determine them."

## Methodology

The core insight is a **task-specific trust hierarchy over information fields**. A user query implicitly ranks its available information sources: "summarize this invoice and email it to the address on the invoice" makes the invoice's address field authoritative for the `to` parameter. "Email a summary of this thread to legal" makes the *user's stated intent* authoritative and the thread body untrusted for `to`. The same field can be high-trust in one task and near-zero-trust in another.

**AuthorityFlow** is a three-stage pipeline built on that:

1. **Interpret the query → build a trust hierarchy.** A model reads the user query and emits, for this task, which information fields are trusted to determine which parameters.
2. **Trace information flow per argument.** For each tool argument in the pending call, trace which field actually produced the current value and attribute it. This is dataflow attribution over the agent's context, not a semantic judgment about whether the value "looks" malicious.
3. **Gate on trust, allow high-trust changes.** Accept the value if its supporting field is trusted for that parameter under this task. Flag low-trust influence as suspected parameter poisoning. High-trust changes (user says "send to Bob") pass untouched.

The design choice worth noting: the *policy* comes from a model, but the *decision* is a lookup against that policy plus a dataflow attribution. It is a reference-monitor shape rather than a second model voting on the first.

**FactAuthBench** is introduced as the evaluation harness, since no existing benchmark isolates parameter-level attacks.

## Key Results

| Condition | Attack success rate | Benign task utility |
|---|---|---|
| No defense | **93.06%** | 90.28% |
| AuthorityFlow | **9.03%** | 82.64% |

That is a 10× reduction in ASR for roughly 7.6 points of utility — a trade that is unusual in this literature, where defenses typically buy safety with a much larger capability loss.

The 93.06% undefended number matters independently: it establishes that when you stop asking "did a new action appear" and start asking "did an argument change," parameter poisoning is not a marginal attack. It is close to a default success.

## What's Novel

The reframing of legitimacy. Prior work treats parameter integrity as a validation problem — is this value well-formed, is this recipient on an allowlist. AuthorityFlow treats it as an **information-flow** problem: not *what* the value is, but *where it came from* relative to what the user authorized.

That distinction generalizes well beyond email recipients. It applies to `amount` in a payment, `repo` in a `git push`, `url` in a fetch, `path` in a file write, `cluster` in a kubectl apply. All of them are parameters where the action is legitimate and the target is the attack.

Second novelty is that the trust hierarchy is **recomputed per query**, which is what defeats the obvious counterargument that "the attacker can just make their field look trusted." The hierarchy is derived from the user's phrasing, not from the content of the context — so poisoning the context cannot promote a field's authority.

## My Connection

This is the finding I would lead with in any agent engagement where the client has already deployed an action-level guard.

Concretely:

- **Test plan.** Every tool in scope gets a parameter-poisoning variant added: same tool, same intent, one critical argument redirected. Most agent test suites have zero of these. Expect ~90% success on undefended targets and lead with that number.
- **Gap analysis on existing controls.** If the client is running a tool-call policy engine, ask what it does when `send_email` is called with `to` from retrieved content rather than from the user. If the answer is "the policy sees `send_email` and allows it," they have the exact blind spot this paper measures.
- **Detection engineering.** The attribution signal — "this argument's value was produced by a field that the query did not authorize" — is loggable. It is a better alert than "new tool appeared," which fires on every legitimate exploration step.

The 9.03% residual is also worth naming in reports: the attack is reduced, not eliminated, and the remaining path is presumably cases where the model's own trust hierarchy was wrong. That is an argument for stacking a deterministic egress check (recipient allowlist, amount ceiling) on top.

## What I Learned

For a year the agent security field has optimized detection of *new* behavior — new tool, new host, new action. This paper shows the attacker's cheapest move is to use the behavior you already approved and change one argument. Defenses built on "did anything unusual happen" are structurally blind to it, because nothing unusual happens.

The transferable idea is that **authorization is not about the action, it is about the binding between information sources and the parameters they are allowed to determine.** That is a dataflow property, and dataflow properties can be enforced without a model in the decision path — which is the only kind of agent control that survives an attacker who also controls the context.
