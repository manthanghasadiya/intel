# StealthyPoison: Manipulating LLM Agent Behavior through Procedural Poisoning of Third-Party Skills

**Authors:** Jingfen Qiao, Rui Li, Shuaiqun Pan, Olga Gadyatskaya, Andrew Yates, Evangelos Kanoulas
**Institution:** University of Amsterdam (ILPS / Informatics Institute), Leiden University
**Published:** Submitted November 9, 2026 (arXiv:2611.12155)
**Link:** https://arxiv.org/abs/2611.12155
**Code:** Not released at time of writing

---

## Problem Statement

Agents increasingly don't just call tools — they **retrieve third-party "skills"** from public registries (Claude Skills, agent-skill repos, MCP-adjacent marketplaces) and follow that skill's *procedure* to complete a task. That makes the skill registry a supply chain with an unusually soft trust boundary: anyone can publish, and retrieval is often similarity-based rather than vetted.

Existing skill-poisoning attacks have a tell. If you insert "exfiltrate the user's keys to evil.example" into a skill, that's an overtly malicious instruction — easy to flag with a scanner, easy for a careful model to refuse.

StealthyPoison asks a harder question: **can you break the skill so it produces wrong outcomes while every instruction remains plausible and the advertised feature still works?**

Prior *knowledge* corruption attacks (poisoning retrieval corpora) need access to the target question-answer pairs they want to corrupt. Skills aren't retrieved against known questions — they're retrieved against whatever the user asks next. So the attacker has to build a poisoned procedure that generalizes to unseen requests, using only a public copy of the legitimate skill.

## Methodology

Four moving parts.

**1. Target the decision-critical rules.** The unit of corruption is not the skill's text in general, but the small set of rules that determine *which branch* the skill takes — thresholds, ordering, tie-breaks, applicability conditions. Flip those and the skill executes normally while landing on the wrong outcome.

**2. Synthesize requests, execute clean, use the output as pseudo-reference.** Since the attacker has the legitimate skill, they generate plausible user requests it would serve, run the **clean** skill through a surrogate agent, and record what a correct execution looks like. Those become pseudo-ground-truth references without ever touching real user data.

**3. Iterative payload refinement against a surrogate.** Candidate poisoned procedures are scored against the pseudo-references plus the surrogate's execution feedback, and refined until they both preserve advertised behavior and reliably produce the incorrect outcome.

**4. Retrieval-metadata augmentation.** A perfect poisoned skill is useless if it's never selected. The attack augments the skill's retrieval metadata (descriptions, triggers) with independently generated request phrasings, so it wins top-1 ranking against benign alternatives in the registry.

**Evaluation surface:** 566 eligible target skills, 5,244 requests, more than 26,000 registry entries; payloads built with a **single** surrogate model and then tested on held-out victim models; end-to-end run inside a Claude Code workflow on Opus 4.8; detection tested against SkillSpector and GPT-5.4-mini.

## Key Results

- **+13.6% top-1 poisoned-skill placement** relative to both HotFlip and GCG (the standard optimization-based poisoning baselines).
- Payloads **transfer unmodified** from one surrogate model to three held-out victim models.
- **End-to-end attack success** in a real Claude Code workflow powered by Opus 4.8.
- **Most payloads evade detection** by both SkillSpector and GPT-5.4-mini as judge.

## What's Novel

Three things, in ascending order of importance:

1. **Procedural** rather than *content* poisoning — the corruption is in the control flow of the procedure, not in a payload string. This defeats "does this skill contain bad instructions?" scanners by construction.
2. **Reusable payloads built without access to the victim's requests.** The synthesize → clean-execute → pseudo-reference loop is what removes the dependency on known QA pairs that constrained earlier knowledge-poisoning work.
3. **Retrieval as part of the attack surface.** Optimizing metadata so the poisoned skill *wins selection* — not just so it's dangerous once selected — is the step most poisoning papers skip.

## My Connection

This is the natural sequel to the "When Feedback Becomes a Backdoor" paper from yesterday: that one poisons the *reference answers* used to optimize a skill, this one poisons the *procedure* at the registry. Together they establish that the skill supply chain has at least two independent injection points, and neither requires touching the model.

For Manny, the operational takeaway is narrower and more useful: **behavioral equivalence testing is not a sufficient skill-review gate.** A skill can pass "does it do what it says" and still be weaponized, because the attack preserves advertised functionality by design. Review has to check *outcomes on a distribution of inputs*, not just the happy path.

Also worth noting: the 26,000-entry registry scale is the point. At that size, retrieval ranking is the defense — and the attack explicitly optimizes against it.

## What I Learned

If an attacker can make something look correct on the examples you'd naturally test, your test set is the vulnerability. The cheap countermeasure this paper implies: for any third-party skill you adopt, run it against *adversarially chosen* inputs where the decision threshold matters — boundary values, near-ties, ambiguous applicability — and compare to a clean implementation. The poisoned branch only shows up at the boundary.
