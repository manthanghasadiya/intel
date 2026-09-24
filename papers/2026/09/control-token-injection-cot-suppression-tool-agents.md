# Control-Token Injection Suppresses Chain-of-Thought and Defeats Reasoning-Based Oversight in Tool-Using Agents

**Authors:** Muhammad Usama, Khair Un Nisa, Summer Yeoreum Jung | **Published:** Sep 23, 2026
**Link:** https://arxiv.org/abs/2609.27542
**Code:** Not stated in the listing
**arXiv ID:** 2609.27542v1

---

## Problem Statement

The safety of a tool-using language-model agent is usually treated as a property of the *model alone* — you red-team the model, you read its chain-of-thought, and you trust that a refusal or a visible reasoning trace means the agent is safe. This paper argues that framing is wrong: safety is a *joint property* of the model **and** the software around it — the chat-template renderer, the tool-call parser (the "decoding harness"), and the runtime that decides whether to execute a generated call. The authors show both halves are attackable from untrusted input, and that "reasoning-based oversight" (monitoring the model's CoT) can be silently switched off.

## Methodology (technical)

- Target: the released **gpt-oss-20b** reasoning model running in its **published tool sandbox**.
- Attack primitive: append a **single string of the model's own channel-control tokens** to a user message. This makes the tokenizer/renderer produce a reasoning turn that is already marked *complete*, so the model emits **no chain-of-thought** and proceeds straight to a tool call.
- Test set: **40 tasks** the model already completes.
- Monitoring evaluation: a **rule-based monitor** and a **cross-family LLM monitor**, run over both plain and forged trials.
- Harness comparison: a **truncation-tolerant regex** parser vs. a **strict** parser, plus **two parsers shipped for the Gemma agent**, compared on identical greedy generations.
- Generality: the suppression is delivered **indirectly**, and the attack's dependence on the **chat template** is characterized across **two more reasoning models**.

## Key Results (with numbers)

- Across the 40 tasks, the **reasoning channel falls from a mean of 52.5 tokens to zero on every trial**, while the `http.post` still fires **on every trial**.
- Both monitors detect the unsafe request on **all plain trials and zero forged trials** — i.e., the oversight signal disappears exactly when the attack is active.
- On overtly malicious requests, the attack converts **39.6%** of the model's refusals into **completed exfiltrations**.
- Whether an *identical* generation fires is decided by the **parser, not the model**: on identical greedy generations, the two Gemma parsers fire on **all 24 trials** and **none**, respectively.

## What's Novel

- Reframes agent safety as a property of the **model + harness + parser** jointly, with controlled full-precision evidence.
- Demonstrates a **tokenizer-level attack** that suppresses CoT *and* disables the monitors that depend on it, without any jailbreak of the model's values.
- Isolates the **parser** as an independent security-critical component: same model output, opposite outcomes depending on regex strictness.

## My Connection (to Manny's work)

This is direct ammunition for agent-harness red-teaming. Any harness whose safety relies on (a) reading CoT and (b) a hand-rolled tool-call parser is exposed to both halves of this attack. Concrete tests Manny can run today: append channel-control tokens to inputs and check whether CoT goes to zero while a sensitive tool still fires; fuzz the parser with truncated/missing closing tokens and diff the fire/no-fire behavior; and confirm monitors still catch a forged turn.

## What I Learned (plain English)

You can't trust "the model refused" or "I can see its reasoning" as evidence of safety, because both signals live in software you control but didn't audit. A few special tokens from the model's own vocabulary can make it think it already finished reasoning — so it acts with no visible thinking — and a sloppy regex in the tool-call parser can be the difference between "request dropped" and "exfiltration completed." Safety has to be enforced where the action actually happens, not inferred from what the model says.
