# Signed, Sealed, Leaked: Trust Beyond Signature for MCP Toolchains

**Authors:** Yichiet Aun (Universiti Tunku Abdul Rahman), Mohammed A. Alhartomi (University of Tabuk), Jing Jie Tan (National University of Singapore), Waqas Nadeem (UTAR), Adeb Salh (UTAR), Shankar Karuppayah (Bitranger / CYRES, Universiti Sains Malaysia), Sin Chee Tan (ABLEs)
**Published:** November 4, 2026
**Link:** https://arxiv.org/abs/2611.06585
**Code:** Not released at time of writing
**Subjects:** cs.CR, cs.AI, cs.MA
**Length:** 24 pages, 5 figures, 15 tables

---

## Problem Statement

The MCP ecosystem's security story has converged on two controls: **publisher signatures** (you know who shipped the tool) and **runtime policy** (a gateway decides which destinations a tool may talk to). Neither is a confidentiality control.

A signed tool can be perfectly authentic and still be malicious — its advertised function works as documented, and it quietly embeds your secrets in an otherwise legitimate outbound request to a domain the policy already allows. Signature verification passes. Destination allowlisting passes. The data leaves anyway.

Meta's MCPGuard-Dynamic is the current state of the art for this class of runtime defence, and the paper's first contribution is to break it: in the same run, it blocks a *direct* exfiltration attempt and still fails to stop the *policy-allowed-destination* variant in every trial.

The deeper argument: as long as a tool's code executes inside the same address space as the data it can reach, any static or behavioural check is playing a losing game. You are trying to predict what arbitrary code will do with secrets it can already read.

## Methodology

**Attack construction.** A legitimately signed MCP tool that performs its advertised function, while embedding sensitive data into a request directed at a policy-allowed destination. No policy violation at any layer — this is the point.

**The proposed defence: "germline execution."**

- A **germline process** is started once, containing no sensitive data at all. It is the template.
- Every tool invocation runs in an **ephemeral fork** of that germline — a copy that has never seen any tenant data.
- Before the copy runs, it receives **only the data authorised for that invocation's destination**.
- The tool's implementation is neither inspected nor trusted. The bound is structural: malicious code cannot disclose what its copy never received.
- The germline is never mutated, so one invocation's state (and one invocation's leak) cannot persist into the next.

This is a **capability-by-construction** framing rather than a detection framing. The security property is stated as *sink noninterference*: no execution of a signed release can cause data at sink A to influence observations at sink B unless that flow was authorised.

**Two-part proof.**

1. **Sink noninterference for every signed release**, under explicit assumptions on copy confinement and germline data-freedom. The proof holds regardless of what the tool does — this is the part that makes it interesting.
2. For releases that are **deterministic within the replay model**, a verified witness establishes a flow violation when one occurs. So you get soundness universally and a decision procedure on the deterministic subset.

**Evaluation.** 153 attack runs, comparing germline execution against baseline defences and an ablation. Plus a performance measurement of fork-per-invocation against process restart.

## Key Results

| Metric | Result |
|---|---|
| Unauthorised disclosure under germline execution | **0 of 153 runs** |
| Baseline / ablation disclosure rate | **5.9% – 74.5%** of runs |
| MCPGuard-Dynamic, direct exfiltration | Blocked |
| MCPGuard-Dynamic, policy-allowed destination | **Failed in every trial** |
| Invocation latency (single copy) | **10–49 ms** |
| Speedup vs. process restart per invocation | **2–28×** |

The performance result matters as much as the security one. The obvious way to guarantee a clean state is to restart the tool process for every call — which is too slow for interactive agents. Forking from a data-free template gets you the same isolation at 10–49 ms, so the defence is deployable rather than theoretical.

## What's Novel

1. **Reframes MCP tool trust as an information-flow problem, not an authentication problem.** Signatures and destination allowlists answer "who" and "where," never "what data." The paper shows those two answers are jointly insufficient, with a working signed-and-leaking tool.
2. **Breaks the leading commercial-grade runtime defence** (MCPGuard-Dynamic) on the evasion that actually matters — the one that stays inside policy.
3. **Germline execution** is a port of OS process-isolation primitives to the agent tool-call layer, with a noninterference proof rather than a benchmark-only claim. The "data-free template, per-destination data grant" pattern is a genuinely reusable primitive.
4. **Honest about scope.** The proof requires copy confinement and germline data-freedom as assumptions. Those are the real engineering problems, and the paper does not pretend otherwise.

## My Connection

This is the cleanest articulation yet of a point that keeps surfacing in the MCP security literature: **every control that operates on the tool's description or the tool's traffic is downstream of the tool's code.** Prior digests have covered poisoning at the description layer (MCPVenom, 22.9% ASR, zero intentional framework defences) and parameter-layer attacks. This paper moves one level deeper — the code is legitimate, the metadata is clean, the destination is allowed, and the leak still happens.

For red-team work, the attack is directly reusable: any signed MCP tool with a legitimate reason to call an allowed external endpoint is a candidate exfiltration channel that passes review. For defence, "germline execution" is the first proposal I've seen that bounds disclosure *without* requiring you to classify the tool as malicious — which is the correct shape for a supply chain you cannot audit.

Practical takeaway for Manny's own agent stack: if a tool server is co-resident with credentials, signature verification is not buying you confidentiality. The question to ask is "what data does this invocation actually need to see," not "is this publisher trusted."

## What I Learned

Signing a tool tells you who wrote it. It tells you nothing about what it will do with your data once it runs. And you cannot fix that by watching the tool's network traffic, because a smart attacker will send the data somewhere you already said was fine.

The fix that works is almost boring: never let the tool run in a process that has your secrets in the first place. Start a clean process with nothing in it, fork it for each call, and hand the fork exactly the data that one call is allowed to touch. If the code turns malicious mid-run, it has nothing to steal — not because you detected it, but because the data was never there.

It's the same idea as giving a contractor a key to one room instead of the building. You don't have to trust the contractor.
