# When "Do Not" Is Not Deny: Security Rules in CLAUDE.md vs Built-In Controls

- **Link:** https://arxiv.org/abs/2608.23550
- **Authors:** Ting Yan
- **Institution:** Not stated in abstract
- **Published:** 2026-08-24
- **Code:** Not mentioned

## Problem Statement (plain English)

Developers write security rules into CLAUDE.md ("do not read ~/.ssh", "never delete files") and assume the agent is constrained. But there are two very different mechanisms at play: a natural-language instruction that the *model interprets*, and Claude Code's built-in `deny`/permission controls that *block the action before it happens*. This paper measures how often a written security rule actually has a built-in control behind it. Spoiler: almost never.

## Methodology (technical)

- Corpus: **481 public CLAUDE.md files**.
- An LLM extracted candidate security rules and matched them against Claude Code's documented built-in controls (permission rules, modes, sandbox).
- **Two security practitioners independently** verified a sample of matches, blinded to the model's answers and each other's labels.
- Matching was evaluated under **multiple strictness standards** (from loose to exact control match).
- A manual review of complete files measured how much of the rule population the extraction method captured.

## Key Results (with numbers)

- Only **~4–16%** of retrieved security rules had a matching built-in control, depending on match strictness.
- Under the **strictest standard: 4.4%** (95% CI: 2.6–6.7%); annotators agreed closely on which rules had matches.
- Extraction captured **66.3%** of eligible security rules in complete files — so even accounting for missed rules, the enforcement rate is dismal.
- Root issue: **CLAUDE.md is a write-only channel** — a developer writes a rule and gets zero feedback on whether any control will enforce it.
- Two hidden rule classes: rules a permission/mode/sandbox *can* enforce, and rules that are *left entirely to model interpretation*.

## What's Novel

- First quantitative measurement of the gap between written agent policy and enforced agent policy, on real-world files at scale.
- Distinguishes the two enforcement mechanisms (interpretive vs. built-in) and shows they are not interchangeable.
- Frames the problem as a **feedback/UX failure**: the tool gives no signal that a rule is aspirational.

## My Connection (to Manny's work)

Any agent-security assessment Manny runs should treat CLAUDE.md-style policy files as *aspirational*, then verify what a permission layer actually enforces — this paper supplies the baseline: expect >90% of written rules to be unenforced. It also motivates tooling that diffs written rules against available built-in controls and reports the enforcement gap, which is a shippable audit product for MCP/Claude Code deployments.

## What I Learned (plain English)

- "Do not X" in an agent config file is a *request to the model*, not a control — and models are unreliable interpreters of their own constraints.
- Enforcement needs to happen **before** the action (deny), not after (interpretation).
- The biggest fix is feedback: config systems should tell you "this rule is not backed by any control" the moment you write it.
- Auditors should never count policy text as security; only mapped, testable controls count.
