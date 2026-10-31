# Poisoned Skills: Propagating Jailbreaks Through Agent Markdown Files

| | |
|---|---|
| **Authors** | Kashyap R Manjusha, Murali Emani |
| **Institution** | Argonne National Laboratory |
| **Published** | October 27, 2026 (submitted); announced October 30, 2026 |
| **Venue** | arXiv preprint |
| **Link** | https://arxiv.org/abs/2610.37738 |
| **Code** | Not stated in the listing |
| **Subjects** | cs.CR (primary), cs.AI |

---

## Problem Statement

Modern agent frameworks let users hand the agent a Markdown file containing the system prompt, tool descriptions and skill definitions. The framework reads that file during initialization and splices it into the model's context. This is the standard configuration path for Claude-style skills, Cursor rules, and most open agent harnesses.

That mechanism is also an injection channel. If the configuration file is the system prompt, then whoever controls the file controls the system prompt. If that file came from a marketplace, a fork, a copied gist, or a colleague's repo, the person who wrote it has partial authority over the model's behavior — and it arrives pre-trusted, at initialization, before any user turn.

The paper asks a narrow, testable version of this: can a jailbreak embedded in a Markdown skill file propagate through an agent and induce prohibited behavior?

## Methodology

**Agent.** A LangChain-based question-answering agent whose system prompt is assembled from a user-provided Markdown file. This is the minimal faithful reproduction of the real configuration path — no exotic harness, no privileged access, just the documented feature.

**Payload.** A jailbreak combining two components:
1. **Persona manipulation** — reframe the model into a character for whom the constraint does not apply.
2. **A GPTFUZZER-based attack** — a fuzzed, model-agnostic adversarial suffix/prompt evolved for transfer.

**Evaluation set.** 270 forbidden questions spanning nine harm categories, giving coverage across policy areas rather than a single failure mode.

**Models.** Seven contemporary open-weight models.

**Conditions.**
- **Baseline:** benign system prompt, same questions — establishes refusal behavior.
- **Injected:** the same system prompt with the jailbreak embedded in the skill file.
- **Obfuscated variants:** two payload strategies tested against the plain-text injection.

**Obfuscation strategies.**
1. **Zero-width character stealth payload** — invisible characters woven through the instruction to evade lexical filters.
2. **Phonetic payload** — grapheme-to-phoneme transformation, so the instruction survives text normalization and string matching.

## Key Results

**Baseline refusal is near-total.** Under a benign system prompt, mean attack success rate is **0.5%**. The models, left alone, refuse.

**The skill file destroys that.** The injected skill file raises mean ASR to **45%**, with a peak of **68.9%** on an individual model. That is roughly a **90x** increase in mean attack success, achieved purely by controlling a file the framework was designed to trust.

**Obfuscation does not help.** Neither variant beat plain text:
- Phonetic variant: mean ASR fell to **30%**.
- Zero-width character variant: did not outperform plain text either.

**Vulnerability tracks the model, not the payload trick.** The authors conclude that susceptibility to configuration-based jailbreaks depends more strongly on the underlying model than on the specific obfuscation technique.

## What's Novel

1. **The channel, named precisely.** Prior injection work targets retrieved content, tool output, or user turns. This targets the *initialization path* — the file that becomes the system prompt. It is the highest-authority input in the entire context and the least scrutinized, because it is configuration rather than data.

2. **A clean causal delta.** 0.5% → 45% isolates the effect of one variable: the content of the skill file. No confounds from retrieval, no multi-turn setup.

3. **A negative result that matters.** Obfuscation underperforms plain text. That is worth as much as the positive finding, because it tells defenders not to build lexical filters, and tells attackers not to waste effort hiding the payload.

4. **The model-dependence claim.** If susceptibility is a property of the model rather than the encoding, then model selection is a security control and prompt-filtering is not.

## My Connection

**For offensive work.** A skill file is the cleanest pretext in the agent stack. It is expected, it is user-supplied by design, it is reviewed as documentation rather than as code, and it lands with system-prompt authority. Manny can demonstrate full policy bypass with a single Markdown file and no exploit chain — the same shape as the "official extension trojanized on a marketplace" story, but at the instruction layer where scanners do not look.

**For defensive work.** The mitigation is not detection, it is provenance and pinning. If the skill file is the system prompt, it needs the same review and supply-chain controls as code that runs. Concretely: pin skill files by commit hash, diff them on update, and treat any change to a `SKILL.md` as a code change requiring review.

**Why obfuscation fails, practically.** Plain text works best, which means a reviewer reading the skill file is the only control that can catch it. Automated lexical filtering is provably the wrong investment here — consistent with the companion paper's finding that scanners catch under half of skill-level problems.

**Pairing.** Read alongside *When Skills Tell Agents Not to Ask* (2610.37734): that paper measures the benign, accidental version at 88,000 skills; this one demonstrates the deliberate, adversarial version. Same file, same trust assumption, two failure modes.

## What I Learned

The finding that stuck with me is that obfuscation made things worse. I would have assumed an invisible zero-width payload beats plain text against any filter. Instead plain text wins outright, and the phonetic variant actively costs 15 points of ASR. The lesson generalizes: when the payload is delivered through a trusted channel, hiding it is unnecessary and counterproductive. The channel's trust is the whole exploit.

The second lesson is about where the system prompt actually lives. Everyone treats "system prompt" as something the developer writes and the user never touches. The moment frameworks let users supply a Markdown file, that stopped being true — and the security model did not update. The 90x jump from 0.5% to 45% is the cost of that gap: we built a configuration feature with system-prompt authority and shipped it with documentation-review rigor.

Third: model choice dominates payload engineering. If susceptibility varies more across the seven models than across the three payload encodings, then all the energy spent on clever jailbreak strings is misallocated relative to just picking a model with better configuration robustness — or, better, not feeding it untrusted configuration at all.
