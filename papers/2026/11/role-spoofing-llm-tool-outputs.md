# Understanding and Mitigating Role Spoofing in LLMs

**Authors:** Ezra Edelman, Benjamin Edelman, Surbhi Goel, Eran Malach
**Institution:** not listed on the arXiv record
**Published:** Submitted Oct 29, 2026 · announced in the Fri, Oct 30, 2026 cs.CR batch
**Link:** https://arxiv.org/abs/2610.40396
**Code:** none linked
**Venue:** none stated (preprint)

---

## The Problem, Plainly

Prompt injection usually gets described as "untrusted text tells the model to do something." This paper isolates a nastier variant: untrusted text makes the model believe **the user said it**.

The paper builds on an earlier observation (Ye et al., 2026) that LLMs suffer *role confusion* — they infer who said what from **how the text reads** rather than from the labels attached to it. So a malicious webpage fetched by an agent, or a tool output, can write in the user's register and get treated as the user. The authors call this **role spoofing**, and it survives a lot of the defenses that work against ordinary injection.

## Methodology

Three parts, in order:

1. **Cue decomposition.** They ask how a model tracks "who said what" and find two independent channels:
   - **Content cues** — semantics and writing style (imperatives, first person, urgency).
   - **Structure cues** — control tokens, delimiters, formatting, the chat template itself.

2. **Attack construction.** They build role-spoofing attacks that combine *fake structure with fake content* — i.e. the attacker forges both the formatting that marks a speaker turn and the style of the speaker they want to impersonate. Combined attacks are far more effective than either channel alone.

3. **Mitigation, and one negative result.** In a toy setting they adversarially train the model to ignore content-based injections. It works against content attacks — and leaves the model fully exposed to structure-based attacks. They then propose **linear probes trained to predict speaker roles in a conversation** as a detector, and show the probes generalize across different settings.

## Key Results

- LLMs are **highly susceptible** to role spoofing when fake structure and fake content are combined (qualitative + toy-setting measurements; the abstract does not give a single headline ASR number).
- **Adversarial training on content cues does not transfer to structure cues.** This is the finding that matters most.
- **Linear probes for role prediction detect role spoofing effectively across settings**, outperforming the behavioral defenses tested.

The negative result also supplies the paper's best explanation for a puzzle practitioners keep hitting: frontier models that are demonstrably more resistant to standard injection benchmarks still get spoofed, because the hardening was applied to the channel the benchmark tested.

## What's Novel

- **Naming and isolating the attack class.** Role spoofing is what you get when injection targets the model's attribution of authority instead of its instruction-following. That's a different failure with a different fix.
- **Separating content cues from structure cues empirically**, and showing defenses trained on one don't cover the other.
- **A probe-based detector on internal representations** rather than another input filter — checking the model's own role state instead of pattern-matching the input.

## My Connection

This is the missing half of the MCP tool-poisoning story. Most MCP work focuses on tool *descriptions* steering the agent (Log-To-Leak, poisoned skills). Role spoofing covers the case where a tool's **output** writes itself into the conversation as the user — the difference between "the tool told the agent to exfiltrate" and "the user asked the agent to exfiltrate," which is exactly the distinction every approval prompt and policy gate is built on.

Practical angles:

- **Attack tooling.** A reusable role-spoofing payload template for tool outputs — forge the chat-template structure, mimic the operator's register — is a clean addition to an agent red-team suite, and it evades content-filter defenses by construction.
- **Detection.** Linear probes over the model's residual stream are implementable in a harness; this is one of the few agent-security papers whose defense you can actually wire into a runtime rather than a filter.
- **Content.** "Your injection defense was trained on the wrong channel" is a strong, defensible post with a reproducible negative result behind it.

Reads well alongside SEAL (arXiv 2610.39758): SEAL tries to delete the injected span, this paper explains what happens when a span survives and impersonates authority instead.

## What I Learned

Models don't reliably know who is talking to them — they guess from formatting and tone, and both are attacker-controlled. And a defense can look like it works because the benchmark attacked only one of the two channels the model actually uses.
