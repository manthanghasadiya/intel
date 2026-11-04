# MCP Tool Poisoning in Agentic AI Frameworks: A Cross-Framework Measurement Study

**Authors:** Farooq Shaikh, Biagio Montaruli
**Venue:** AISec '26 (19th Workshop on AI and Security), co-located with ACM CCS 2026, The Hague
**Published:** November 2, 2026
**Link:** https://arxiv.org/abs/2611.03555
**Code:** `mcpvenom` — open-source attack driver released with the paper
**Subjects:** cs.CR, cs.AI

---

## Problem Statement

MCP is the de facto connector between agent frameworks and external tool providers. It carries attacker-controlled content into model context with no validation: tool descriptions, JSON-schema field names and field descriptions, and tool responses. A malicious or compromised MCP server can therefore put instructions in front of the model through a channel the user never sees.

The prior literature measures the wrong thing. Existing benchmarks score **model compliance** — did the model repeat the poisoned instruction, did it say it would exfiltrate — but never confirm that a real tool call executed and real data left. A model can "comply" in text and take no action; it can also take the action silently and never mention it. Neither is captured by compliance scoring. That gap means the field has been quoting ASR numbers that may not correspond to any actual harm.

## Methodology

- **Root-cause taxonomy:** 25 classes, consolidating prior classifications into a single ontology (so results can be compared across papers).
- **`mcpvenom`:** an open-source harness that instantiates a real MCP server, injects poisoned metadata, connects a real client framework, and observes the calls that actually get dispatched.
- **Susceptibility matrix:** 9,720 cells = **12 frameworks × 18 models × 45 attack scenarios**.
- **Dual oracle:** every scenario is scored twice — (1) did the agent dispatch an attacker-defined tool call, and (2) did a canary HTTP endpoint actually receive the secret. The second oracle is what makes the result a harm measurement rather than a compliance measurement.
- **Code-level audit:** manual review of all twelve frameworks for any intentional anti-poisoning control.
- **Disclosure:** five coordinated reports across four vendors.

## Key Results

| Metric | Value |
|---|---|
| Overall attack success rate | **22.9%** (2,113 of 9,720 cells) |
| Of vulnerable cells: dispatch attacker tool call | 64.9% |
| Of vulnerable cells: contact attacker endpoint | 23.0% |
| Of vulnerable cells: **verifiable secret exfiltration** | 12.1% |
| Framework clustering | nine core frameworks at 23–26%, χ²=3.30, p=0.91 |
| Model ASR range | **0.4% – 49.7%** |
| Claude Opus 4.8 | vulnerable in 8 of 530 cells, with 223 explicit refusals |
| Regex description scanner | rejects 20 of 45 scenarios, 7.2% false-positive rate |
| Intentional framework defenses found | **0 of 12** |
| Coordinated disclosures rejected | **5 of 5** (four vendors) |

Three findings carry the paper:

1. **Framework choice is statistically irrelevant.** Nine core frameworks land within three percentage points of each other at p=0.91. There is no "more secure MCP client." The variance is in the model.
2. **Resistance is a model property, and it is bimodal.** 0.4% to 49.7% is a hundredfold spread. Opus 4.8's 8/530 with 223 refusals suggests refusal behavior, not architectural safety — the model declines out loud, and on rare occasions acts anyway.
3. **Metadata scanning is a partial control.** The scanner catches 44% of scenarios at 7.2% FPR. Useful as a filter, useless as a boundary. And the incidental mitigations that do exist in frameworks are accidents of design, not defenses — which means a refactor can remove them.

## What's Novel

The dual oracle. Prior work measured what the model *said*; this measures what the network *received*. The distinction between 22.9% compliance and 12.1% verified exfiltration is the difference between a scare number and a risk number, and it should reset how the field reports MCP ASR.

Second novelty is the negative result on frameworks. A cross-framework study that found meaningful differences would have launched twelve vendor blog posts. Finding p=0.91 is more valuable: it tells defenders where the leverage actually is, and it tells buyers that "we mitigate tool poisoning" is not a differentiator.

Third is the disclosure record: 5/5 rejected. That is a finding about the ecosystem's response capacity, not just about the bug class.

## My Connection

This is directly load-bearing for agent red-team engagements. Three concrete uses:

- **Report framing.** When I write "the agent executed the injected tool call," I can now cite a harm-confirmed measurement methodology instead of a compliance benchmark. The dual-oracle pattern (dispatch + canary receipt) is the right evidence standard for any agent finding.
- **Scoping.** If a client asks which MCP client to standardize on for security, the honest answer is "the model is the variable, budget accordingly." That reframes the engagement from procurement review to model evaluation plus egress control.
- **Test harness reuse.** `mcpvenom` gives a starting matrix. 45 scenarios × a handful of models is a tractable engagement scope, and the 25-class taxonomy maps cleanly to findings categories.

The 44%-at-7.2%-FPR scanner result also gives a defensible answer to "can't we just scan the tool descriptions?" — yes, for less than half, and you will page on 7 in 100 benign descriptions.

## What I Learned

Measuring whether a model *says* it will do something is nearly worthless; measuring whether the network *receives* something is the whole game. If a benchmark does not have a canary, it is a survey of model opinions.

The deeper lesson is that the MCP security problem has been misframed as a client-hardening problem. It is a **model-selection plus egress-control** problem. Twelve frameworks, zero defenses, no statistical difference between them — that is not a market that is about to fix itself, and the five rejected disclosures confirm nobody is racing to. The practical implication: assume the poisoned tool call will be dispatched, and put your control where the data leaves.
