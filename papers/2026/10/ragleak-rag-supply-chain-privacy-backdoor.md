# RAGLeak: Supply-Chain Privacy Backdoors in Retrieval-Augmented Generation

**Authors:** Yuchen Chen, Wei Cheng, Yuan Xiao, Junda He, Jieke Shi, PingPing OuYang, Quanjun Zhang, Chunrong Fang, David Lo, Zhenyu Chen
**Institution:** affiliated institutions not listed on the arXiv record (author set spans multiple groups)
**Published:** October 19, 2026 | **arXiv:** [2610.25937](https://arxiv.org/abs/2610.25937) (cs.CR)
**Code:** [anonymous.4open.science/r/RAGLeak-4B67](https://anonymous.4open.science/r/RAGLeak-4B67) (anonymized artifact)

---

## Problem Statement

RAG systems trust their knowledge base. That is the entire premise — you retrieve documents you did not write and hand them to a model you did write, and the output is supposed to reflect your data.

Existing data-extraction attacks against RAG have two failure modes. **Prompt injection** to exfiltrate retrieved documents depends on the generator's instruction-following: harden the model, and the attack dies. **Document-reproduction backdoors** (make the model emit or paraphrase a poisoned document) struggle to balance *fidelity* (do you get the actual text, or a lossy paraphrase?) against *stealth* (does the poisoning survive detection?). High fidelity is conspicuous; high stealth loses the data.

RAGLeak's insight: **decouple the leak from direct textual exposure.** Don't make the model recite the secret. Make it emit something that only looks like noise — and reconstruct the secret offline.

## Methodology

**The backdoor.** A poisoned document is planted in the knowledge base. Normally it is inert. When triggered, it causes the generator to emit a *reversibly obfuscated* transformation of retrieved private documents rather than their plaintext.

**The reconstruction.** The obfuscation is reversible by an auxiliary LLM held by the attacker. The victim's model outputs something that reads as garbled or unrelated content; the attacker collects it and inverts the transformation to recover the original document at high fidelity.

**Why this beats prior work.** The victim-side output never contains the secret in a form that a content filter, a canary, an output classifier, or a human reviewer would recognize. Fidelity is not lost to paraphrase — it is preserved by construction, because the transformation is invertible. Stealth comes from the fact that "the model produced strange text" is not the same signal as "the model produced the secret."

**Evaluation.** Three model families × three application domains, tested against representative detection-based defenses, plus clean fine-tuning and cross-scale/cross-version transfer.

## Key Results

| Condition | Result |
|---|---|
| Attack success rate | up to **100%** |
| Impact on benign utility | limited |
| Under detection-based defenses | at most a **10 percentage-point** ASR drop |
| After clean fine-tuning | backdoor remains effective |
| Across model scales and versions | remains effective |
| Code | public anonymized artifact |

The "survives clean fine-tuning" result is the one to sit with. Fine-tuning on clean data is the standard remediation for a poisoned model, and here it does not remove the behavior — consistent with the trigger living in the retrieval path rather than in a narrowly-learned output pattern.

## What's Novel

1. **Reversible obfuscation as the exfiltration channel.** Prior work asks "can the model be made to say the secret?" This asks "can the model be made to say something only the attacker can decode?" — which moves the leak entirely out of the space that output filters inspect.
2. **Fidelity/stealth tradeoff dissolved.** The usual tension exists only if the leaked artifact must resemble the secret. Invertible transforms mean the artifact can look like anything.
3. **Supply-chain framing.** The poison enters via the knowledge base, not the model or the prompt — a component most RAG deployments treat as data rather than as code, and therefore review far less.
4. **Durability evidence.** Surviving clean fine-tuning, model scale changes and version changes is a much stronger persistence claim than single-model ASR.

## My Connection

The knowledge base is the least-governed artifact in most agent stacks. MCP servers frequently *are* retrieval layers — a "search our docs" tool, a "query the wiki" tool, a vector store behind a filesystem tool. If the retrieval corpus is attacker-influenceable (shared drives, ingested tickets, crawled pages, third-party document feeds), then this class of backdoor is in scope, and the standard defenses — output DLP, prompt-injection guardrails, content classifiers — are aimed at the wrong artifact.

The concrete takeaway for MCP gateway design: **treat retrieved content as an input to be attributed, not just filtered.** Provenance (which document, which ingest path, which uploader) is the control that would catch this, because the poisoned document has to get in somehow. Output-side detection is structurally blind to a channel whose payload is deliberately unrecognizable.

## What I Learned

Every defense assumes a shape for the attack. Output filters assume the leak looks like the secret. Injection defenses assume the attack is an instruction. RAGLeak wins by making the leaked artifact look like neither — it is ciphertext wearing the costume of model noise.

The general lesson: **when a defense inspects content for similarity to a protected asset, an invertible transform defeats it completely**, because "does not resemble the secret" and "is not the secret" are different claims. The only defenses that survive are ones that reason about *where data came from* and *who could have put it there* — provenance and integrity, not resemblance. That is an argument for signing and attributing your retrieval corpus, which almost nobody does.
