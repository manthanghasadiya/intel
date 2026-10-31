# Scan Before You Install: Detecting and Mitigating Local and Composite Vulnerabilities in Skills

| | |
|---|---|
| **Authors** | Parth A. Gandhi, Avital Aviv, Prasanna N. Wudali, Yuval Elovici, Asaf Shabtai |
| **Institution** | Ben-Gurion University of the Negev |
| **Published** | October 27, 2026 (submitted); announced October 30, 2026 |
| **Venue** | arXiv preprint |
| **Link** | https://arxiv.org/abs/2610.37901 |
| **Code** | Not stated in the listing |
| **Subjects** | cs.CR (primary), cs.AI |

---

## Problem Statement

An agent skill bundles natural-language instructions, scripts and reference files that an LLM agent loads for a task. Under the permissions it is granted, a skill can touch the filesystem, the shell and the network. A weakness in a skill is therefore not a weakness in a document — it is a weakness in an execution path.

Two gaps in existing tooling:

**Gap 1 — malicious versus merely broken.** Current scanners hunt for *malicious* skills: prompt injection, exfiltration, obvious payloads. They largely ignore *unintended* vulnerabilities in otherwise benign skills — an unpinned `pip install`, an unquoted shell interpolation, a path built from user input, a permissive `sudo` invocation. Public repositories are full of these, and they were not written by attackers.

**Gap 2 — isolation versus composition.** Scanning one skill at a time misses the multi-skill case. Skill A writes a file; Skill B reads it and passes it to a shell; Skill C's instruction tells B to skip validation. Each skill passes alone. The chain does not.

## Methodology

A six-module scanner pipeline:

1. **Content Extraction** — pull instructions, scripts and reference files out of the skill package.
2. **Local Analysis** — detect unintended vulnerabilities within a single skill.
3. **Composite Analysis** — model dependencies *between* skills.
4. **Validation** — confirm findings rather than emitting raw matches.
5. **Reporting** — surface findings with severity.
6. **Mitigation** — recommend fixes.

**Composite Analysis detail.** The interesting module. It builds a dependency model across skills through three relation types:
- **Shared resources** — one skill writes what another reads.
- **Invocations** — one skill calls another.
- **Instruction-mediated interactions** — one skill's natural-language instruction changes how another's output is handled.

That third relation is what makes this hard and what no static dependency graph captures: the link between two skills exists only in the prose of a `SKILL.md`.

**Evaluation sets.**
- **SkillTrustBench** — a 556-case leaderboard subset.
- **MalSkillBench** — a balanced sample of 1,000 skills.
- **SCR-Bench** — for composite findings.
- **Real-world corpus** — 1,587 skills from the 75 most-starred GitHub repositories, findings confirmed by three human analysts.

**Baselines.** Cisco's Skill Scanner, NVIDIA's SkillSpector, AgentSkillsScanner, SkillSieve, and three SkillsGuard configurations released with CompoSkill.

## Key Results

**Local analysis.**
- **95.90% recall** at **4.22% false positive rate** on the 556-case SkillTrustBench subset.
- **F1 0.947** on the balanced 1,000-skill MalSkillBench sample — beating Cisco's Skill Scanner, NVIDIA's SkillSpector, AgentSkillsScanner and SkillSieve.

**Real-world precision.**
- 1,587 skills from the 75 most-starred GitHub repos.
- Three analysts confirmed **53 of 59 findings** — **89.8% precision** — spread across **23 repositories**.
- PoC attacks were developed for each vulnerability.
- **10 medium- and high-severity cases** disclosed to maintainers.

**Composite analysis.**
- **F1 0.991** on SCR-Bench (**98.7% precision, 99.6% recall**), against **0.816–0.858** for three SkillsGuard configurations.
- Identified **two composite findings on GitHub**, both validated by PoC attack.

**Mitigation.**
- Recommends skill substitution, patches and configuration changes.
- For **47 of 55** GitHub findings, mitigations prevented the PoC attack in **all trials** while preserving tested legitimate functionality.

## What's Novel

1. **Benign-but-vulnerable as a category.** The entire field was oriented at malicious skills. This paper points the scanner at ordinary, well-intentioned public skills and finds real, exploitable weaknesses — 53 confirmed across 23 top repositories.

2. **Composite skill vulnerabilities.** Modeling interactions through *instruction-mediated* links, not just shared files or explicit calls, is the conceptual contribution. It acknowledges that in an agent system the dependency graph partly lives in prose.

3. **Validation as a pipeline stage.** A dedicated module that confirms findings before reporting is why precision lands at 89.8% on real repos rather than collapsing into noise.

4. **Mitigation with a preservation check.** Reporting that 47 of 55 fixes blocked the attack *while keeping legitimate functionality working* is the part most security tooling skips and the part that decides whether a maintainer accepts the patch.

5. **Head-to-head against the incumbents** — Cisco, NVIDIA, and the SkillsGuard line — on shared benchmarks, rather than self-reported numbers.

## My Connection

This is the tool Manny should be running, or rebuilding, today.

- **The scanner is the product.** 95.9% recall at 4.22% FPR on local analysis and 89.8% precision on real GitHub repos crosses the threshold where you can gate a marketplace or a CI pipeline on it. That is a defensible artifact to ship or to compete with.
- **Composite analysis is the differentiator worth copying.** Everyone can grep a skill for `curl | sh`. Almost nobody models Skill A writes → Skill B reads → Skill C disables validation. On SCR-Bench the gap is 0.991 versus ~0.83 — a large margin on exactly the case that matters for real agent deployments.
- **The 23 repositories are a target list.** 53 confirmed vulnerabilities across 23 of the 75 most-starred skill repos is a ready-made responsible-disclosure or content program, and a demo corpus that is already validated.
- **Pair with the measurement paper.** *When Skills Tell Agents Not to Ask* found existing scanners catch at most 46% of oversight-loosening directives. This paper's scanner is the one with the headroom to close that gap — but note it was benchmarked on vulnerability classes, not on oversight directives. Running it against the 88,000-skill loosening corpus is an obvious, unclaimed experiment.

## What I Learned

The distinction that reorganizes my thinking is malicious versus unintended. I had been modeling skills as a supply-chain problem — someone ships a bad package — which leads to a trust-and-provenance answer. This paper says most of the risk is in skills nobody attacked: ordinary code with an unquoted variable, an unpinned install, a `sudo` that was convenient at the time. Trust does not help you there, because the author is trustworthy and the skill is still exploitable.

The composite result is the more unsettling one. A skill that is perfectly safe alone becomes dangerous in the presence of two other skills, and the connection is expressed in natural language rather than in an import statement. That means there is no static analysis of the traditional kind available — you have to reason about what the instructions *mean*, which is why the instruction-mediated relation had to be modeled explicitly rather than derived.

And the mitigation numbers are the tell that this is engineering rather than research theater. 47 of 55 fixes working in all trials *while preserving legitimate functionality* is the only result that gets a maintainer to merge. Security tooling that cannot demonstrate it did not break the thing gets ignored, correctly.
