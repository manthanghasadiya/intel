# Shortcut Sensitivity in Agent-Safety Evaluations

**Authors:** Hasan Demirkiran, Jens Ernstberger
**Published:** 2026-10-15
**Link:** https://arxiv.org/abs/2610.22243
**Subjects:** cs.AI (primary), cs.CR, cs.LG
**Code/Data:** Not linked in the abstract; the paper releases its audit methodology and reports a placebo-controlled screen on a frozen 64-step AgentDojo-Traj cohort.

---

## Problem Statement

Step-level guards decide, before each tool call, whether an agent's next action is safe. Those guards are evaluated on benchmark datasets — but the dataset itself can contain cues that make labels guessable without measuring safety at all. If a dataset's "unsafe" trajectories happen to have shorter shell commands, a guard can score well by learning "short command = unsafe" and never learn the safety construct. Worse, the same cues can leak into guard behavior at inference time. The paper asks a question most safety-benchmark papers skip: **is the evaluation measuring the guard, or the dataset's formatting?**

Three distinct failure modes get separated:

1. **Label predictability** — can you predict the label from a shortcut feature alone?
2. **Observational guard association** — does the guard's output correlate with that shortcut?
3. **Paired decision sensitivity** — if you perturb only the shortcut, does the decision change?

## Methodology

A three-level shortcut-auditing framework applied to three corpora:

- **LinuxArena** — completed agent trajectories (full-task labels).
- **AgentDojo-Traj** — step-level dataset from TS-Bench.
- **AgentHarm-Traj** — step-level dataset with a released guard (TS-Guard).

Measurements performed:

1. **Label predictability.** Fit simple features (e.g., average shell-command length) and measure how well they predict labels. LinuxArena: shorter average shell commands strongly predict trajectory labels.
2. **Fold sensitivity.** Compare mixed-domain cross-validation against domain holdout. AgentDojo-Traj: exact-tool lookup falls from F1 = .901 (mixed-domain folds) to .495 (domain holdout) — the model was memorizing tool identities, not safety.
3. **Weighting sensitivity.** AgentHarm-Traj: released TS-Guard F1 falls from .902 under row weighting to .719 under equal interaction weighting (balanced accuracy rises, showing the two metrics disagree in direction).
4. **Paired intervention with placebo.** Freeze a 64-step AgentDojo-Traj cohort, run seven configurations with repeated interleaved Haiku calls, and perturb only representational details:
   - Rewrite the `<INFORMATION>` marker → blocking drops **8.33 percentage points**.
   - Replace tool names with opaque aliases → blocking rises **11.46 points**.
   - A formatting-only placebo is subtracted from both effects; exploratory 95% ranges exclude zero.
5. **Model-size control.** Repeat in Luna and a deterministic Q8 TS-Guard — the Haiku-sized effects do not reproduce, so sensitivity is model-dependent, not universal.

Critically: all unsafe calls remain blocked throughout. The movement is entirely in **false positives against inherited labels** — i.e., benchmark representation changes measured accuracy without changing actual safety.

## Key Results

| Finding | Number |
|---|---|
| Exact-tool lookup F1, mixed-domain folds → domain holdout | .901 → .495 |
| TS-Guard F1, row weighting → equal interaction weighting | .902 → .719 |
| Blocking change from rewriting `<INFORMATION>` marker | −8.33 pp |
| Blocking change from opaque tool aliases | +11.46 pp |
| Unsafe calls newly allowed | 0 |
| Effect reproduced in Luna / Q8 TS-Guard | No |

## What's Novel

- Separates **label predictability**, **observational association**, and **causal decision sensitivity** instead of treating "the guard scores well" as one number.
- Introduces **placebo-controlled paired interventions** to agent-safety evaluation — perturb representation only, hold behavior fixed, measure the delta. This is the eval equivalent of a randomized trial and is rare in this literature.
- Demonstrates empirically that **benchmark representation changes guard decisions**, not just scores, with confidence ranges that exclude zero.
- Prescribes concrete fixes: domain group holdouts, evaluation-unit metrics, placebo-controlled paired interventions.

## My Connection

This is directly usable in Manny's red-team workflow in two ways:

1. **Guard bypass research.** If rewriting an `<INFORMATION>` marker moves blocking by 8 points without touching the payload, then representational perturbation is a *free* attack primitive against step-level guards — cheaper than prompt injection and orthogonal to it. Systematically fuzzing formatting (marker names, tool aliases, whitespace, JSON key order) is a test suite worth building.
2. **Evaluation integrity.** Any internal agent-safety benchmark being used to sign off on a deployment should be checked for tool-identity memorization (domain holdout) and command-length leakage. A guard that scores .90 on mixed folds and .50 on domain holdout is not a control.

The "unsafe calls all still blocked" caveat matters: this is a false-positive/false-negative asymmetry story, and the practical exploit is degrading a guard until operators disable it, not slipping a payload past it.

## What I Learned

A safety benchmark can be internally consistent, reproducible, and still measure the wrong thing. If a model can guess the label from formatting, then a high score tells you the model learned the formatting. The fix is not a better model — it is a better experiment: hold the behavior fixed, change only the representation, and see if the verdict moves. If it does, the verdict was never about behavior.
