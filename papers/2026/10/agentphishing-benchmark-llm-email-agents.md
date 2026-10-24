# AgentPhishing: A Benchmark of LLM Email Agents Under Adaptive Attack

**Authors:** Justin Tan, Miao Li, Ding Zhao
**Institution:** Carnegie Mellon University
**Published:** October 22, 2026 (announced in arXiv cs.CR new listing Oct 23)
**Link:** https://arxiv.org/abs/2610.31404
**Code:** None public as of publication

---

## Problem Statement

Phishing evaluation has always been built for humans and for classifiers: does the message look malicious? Email agents break that assumption in two directions. First, they read mail at a volume no human matches, so a low success rate still scales. Second — the part this paper targets — an agent does not merely *classify* a message, it *acts* on it, so the security question shifts from "did it detect the phish" to "did it comply with the request."

The authors' argument is that existing phishing corpora are the wrong test set. Published phishing datasets were collected against human judgment or against spam filters, so an agent that has been aligned on them can look safe while remaining wide open to messages written specifically to exploit agent behavior — tool use, urgency framing, authority framing, workflow conventions.

## Methodology

Three components:

**1. The AgentPhishing benchmark.** A task environment where LLM email agents operate over a mailbox with tools, and success is measured on two axes: phishing detection rate (did it flag the message) and malicious-request compliance rate (did it do what the attacker asked). Both matter, and the pair separates "failed to notice" from "noticed but complied."

**2. PhishOpt — adaptive attacker.** Rather than hand-writing emails, the authors optimize *reusable email-generation skills* against an individual target agent, under (a) the fixed benchmark constraints and (b) an explicit realism constraint so generated messages remain plausible business email rather than adversarial gibberish. "Skills" here means persistent, transferable generation procedures — closer to a prompt artifact an operator would keep in a toolkit than a one-shot adversarial string.

**3. Persuasion analysis via Cialdini's principles.** Generated emails are annotated against the six influence principles (reciprocity, commitment/consistency, social proof, authority, liking, scarcity) to identify which levers correlate with agent failure. The authors then use those correlations to construct a *universal* generation skill — one that is not tuned to a specific agent.

That last step is the important methodological move: it converts per-agent red-teaming into a transferable attack artifact, which is exactly how offensive tooling matures.

## Key Results

- Agents that handled pre-existing (conventional) phishing emails safely still failed against behavior-adapted attacks. Safety on the old benchmark carries essentially no information about safety on the new one.
- Across nine agents evaluated on AgentPhishing: **four showed phishing-detection rates below 10%** while simultaneously showing **malicious-request compliance rates above 45%**.
- Persuasion-principle analysis identified the failure-associated strategies, and a universal generation skill derived from them remained effective without per-agent optimization.

The below-10%-detection / above-45%-compliance pairing is the headline number: these agents are not marginally worse than humans at noticing — they are near-blind to the attack class while complying with it nearly half the time.

## What's Novel

1. **Compliance as the primary metric, not detection.** Most prompt-injection benchmarks score "did the model follow the injected instruction." This one scores the *consequence* in a realistic email workflow, with both numbers reported — which distinguishes a detection failure from a judgment failure.
2. **Reusable, optimized generation skills** instead of a static adversarial dataset. The artifact transfers; the benchmark does not go stale the moment models are patched.
3. **Realism as an explicit constraint** in the optimization loop, which forces attacks into the distribution of messages a mail gateway would actually pass.
4. **Persuasion-theoretic labeling** as a diagnostic layer, giving defenders named attack strategies rather than anonymous failing examples.

## My Connection

This is the direct behavioral counterpart to the Phantom ransomware writeup in today's digest (INC / The Gentlemen affiliates running OpenCode + 32 MCP servers, with a human rewriting refusals as "authorized red team assessments"). Both point at the same conclusion from different ends: **the persuasion layer, not the payload layer, is where agent compromise happens.**

Practically, for Manny's work:

- Red-team harnesses should include a compliance axis. An agent that flags a message but then runs `forward_to` on it is compromised, and a detection-only metric scores it as a pass.
- The universal-skill result argues for testing agents against *transferable* attack artifacts, not per-target hand-crafted prompts — otherwise you overestimate safety every time.
- Email is only the first surface. The same PhishOpt-shaped loop applies to calendar invites, Jira tickets, PR descriptions, and — most relevant — MCP tool descriptions, where the "email" is a tool manifest.

## What I Learned

Agent safety evaluation has a generalization problem that looks exactly like the classic one: agents are trained and tested on adversarial examples written by people who were thinking about *other* targets. An attacker who can run an optimization loop against your specific agent gets a different, much more favorable result — and if they can then distill that into a reusable skill, they do not need access to your agent at all to build it.

The uncomfortable operational takeaway: a refusal is not a control. In both the paper and the ransomware case, the attack adapted instead of failing.
