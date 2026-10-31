# When Skills Tell Agents Not to Ask

**A Large-Scale Study of Oversight Directives in Agent Skills**

| | |
|---|---|
| **Authors** | Timofey Sanko, Yuan Tian |
| **Published** | October 27, 2026 (submitted); announced October 30, 2026 |
| **Venue** | arXiv preprint |
| **Link** | https://arxiv.org/abs/2610.37734 |
| **Code / Data** | Not stated in the listing |
| **Subjects** | cs.CR (primary), cs.AI, cs.SE |

---

## Problem Statement

An agent skill is a reusable instruction package — a directory anchored on a `SKILL.md` file — that an agent loads on demand. Skills are copied and forked freely, which means a single line written once by one author reaches every agent that later loads that skill.

Some of those lines remove a checkpoint. They tell the agent to proceed without approval, to pass `--no-verify` and skip the pre-commit hook, to force-push, to keep working without checking in. Each one is a deliberate convenience for the author. Composed across thousands of skills and layered into a single agent context, they become a systematic erosion of exactly the human-in-the-loop guarantees agent harnesses are built to provide.

No prior work had measured this. Everyone measured malicious skills — supply-chain poisoning, prompt injection payloads. Nobody had measured the *benign* kind: the well-intentioned line that just happens to tell the agent to stop asking.

## Methodology

**Corpus.** 1.53 million English skills from GitSkills, a large public aggregator of agent skill repositories.

**Unit decomposition.** Skills are split into units small enough to label: sentences, bullet points, table cells and individual code lines. A single bullet can carry a directive; a whole skill usually cannot.

**Codebook.** The authors built a labelling scheme that classifies oversight directives along three axes:
- **Polarity** — does it tighten or loosen oversight?
- **Mechanism** — permission flags, approval bypass, check skipping, autonomous continuation
- **Scope** — unconditional, or conditioned on a context such as CI

**Human labelling.** 2,097 units labelled by hand to seed and validate the detector.

**Detector.** A trained classifier scales the codebook across the full 1.53M corpus.

**Robustness.** Associations are checked across multiple robustness configurations — notably the comparison against skills with an *equal amount* of oversight language, which isolates loosening from mere verbosity.

**Scanner evaluation.** Open-source skill scanners are then run against the loosening directives to establish how much of this class the current tooling actually sees. An LLM prompted with the paper's codebook is tested as an alternative.

## Key Results

**Prevalence.**
- About **1 in 17 skills** — roughly **88,000 skills** — contains a loosening directive.
- **70.1%** of those directives apply **unconditionally**, not just in CI or a narrow context. The typical case is an unqualified "don't ask me, just do it."

**Mechanism.** Permission flags are the most common vector.

**Correlation with capability.**
- Skills that run shell code, install packages, or invoke `sudo` have **1.7–1.8x the odds** of containing a loosening directive, holding across all robustness checks.
- Crucially, they do **not** compensate with more tightening language. Capability and oversight reduction rise together.

**Propagation.**
- **2.4%** of skill families carrying loosening account for **half** of all copies of loosening-carrying skills.
- **1.1%** of distinct loosening units account for **half** of all loosening occurrences.
- Loosening spreads by copying a few upstream skills, not by independent edits from adopters. Fix the upstream, fix the downstream.

**Tooling gap.**
- Open-source skill scanners catch **at most 46%** of loosening directives, with **precision below 0.4**.
- The LLM behind those scanners, re-prompted with the paper's codebook, catches **61% at precision 0.71**.

## What's Novel

1. **The object of study.** Not malicious skills, but *oversight-loosening* directives — a class nobody had named, let alone measured. This reframes the skill supply chain from a malware problem to a governance problem.

2. **Scale with labelling discipline.** 1.53M skills is a census, not a sample, and it rests on a hand-labelled codebook rather than an ad-hoc keyword list.

3. **The concentration finding.** 2.4% of families producing half the copies is the paper's most operationally useful result. It converts a diffuse 88,000-skill problem into a small patch list.

4. **The capability correlation.** Skills that can run shell code are also the ones most likely to remove oversight, and they do not trade off. That is the opposite of what a risk-weighted design would produce.

5. **A measured tooling gap.** Quantifying that scanners catch under half at under 0.4 precision, and that the LLM underneath them does better with a better codebook, isolates the failure as a *specification* problem rather than a model problem.

## My Connection

This is directly load-bearing for Manny's agent harness work:

- **Audit your own skill tree first.** Any `.claude/skills`, `.cursor/rules`, or custom `SKILL.md` directory Manny maintains should be scanned for the mechanisms the paper names: `--no-verify`, `--force`, `sudo`, unconditional approval bypass, autonomous continuation wording. The paper gives a ready-made taxonomy.
- **The codebook is the deliverable.** A detector built from this codebook beat the LLM inside existing scanners by 15 points of recall and nearly doubled precision. That is a cheap, high-leverage build.
- **The 2.4% concentration is a prioritization gift.** Auditing the top handful of upstream skill families covers most of the real exposure.
- **Red-team framing.** "Benign convenience flags" are a far better pretext for a demo than exotic injection payloads: they are already in the repo, already reviewed, already approved.

## What I Learned

Skills are the first place in the agent stack where the security control and the productivity feature are the same line of text. `--no-verify` exists because hooks are annoying. It also disables the last automated check before code lands. You cannot remove the risk without removing the convenience, which is why it spreads.

The diffusion result is the part I did not expect. I assumed oversight erosion would accumulate from thousands of independent authors each making a small local tradeoff. Instead it concentrates: a few upstream skills get copied and the copies inherit the loosening wholesale. That means this is tractable — not a culture problem across 88,000 authors, but a review problem across a few hundred upstream packages.

And the tooling result reframes what scanners are for. A 46% ceiling at sub-0.4 precision is not a scanner you can put in a CI gate; it is a scanner that produces noise. The fix was not a bigger model, it was telling the model what to look for. Specification, not capability.
