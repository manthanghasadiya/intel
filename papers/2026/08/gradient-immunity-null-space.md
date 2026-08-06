# Gradient Immunity: Null-Space Resistance to Malicious Fine-Tuning

- **Link:** https://arxiv.org/abs/2608.05045
- **Authors:** Yuxuan Huang, Xingyu Zeng, Tianhang Zheng, Chaochao Lu
- **Institution:** Not stated on arXiv abstract page
- **Published:** 2026-08-05
- **Code:** Not stated

---

## Problem Statement (plain English)

Open-weight models are released, then anyone fine-tunes them — including attackers who want to strip out safety training. Existing defenses either assume a "fine-tuning-as-a-service" paradigm (the provider controls the tuning) or ask downstream users to follow safety procedures. Neither works for the real open-source case: a provider releases most weights as trainable while wanting to keep a small safety-critical component protected. Can a provider harden a release so harmful fine-tuning fails — without wrecking the model's utility for honest users?

## Methodology (technical)

The defense is a **Unidirectional Safety Gate (USG)** inserted after the final Transformer layer, composed of:

1. **Null Space Cubic Layer** — during downstream fine-tuning, suppresses/blocks gradients from harmful samples whose hidden states fall in a calibrated *protected region* (a null-space projection that zeroes the harmful direction).
2. **Inverse Adapter** — restores the base model's forward behavior so the gate is invisible at inference for benign inputs.

Calibration: the defender uses a small set of defender-held harmful examples to set a threshold; protection then generalizes to nearby in-distribution harmful samples. This is a *release-time representation-space* defense — the protection lives in the released architecture itself, not in a service workflow.

## Key Results (with numbers)

- Across **six model-dataset settings**, USG keeps post-finetuning attack success rate close to the pre-release level under a fixed release threshold.
- Maintains high safe-pass rates on easier settings (benign fine-tuning still works).
- Exhibits a clearer safety–utility trade-off on unsafe samples from BeaverTails (better than baselines at keeping utility while blocking harmful tuning).

## What's Novel

- First defense aimed at the **partially protected open-weight (PPOW)** release setting — most weights trainable, safety-critical component preserved.
- Representation-space blocking at release time rather than training-time intervention or post-hoc guardrails.
- Gradient-level control: harmful samples are blocked *by direction* (null space), not by content filtering.

## My Connection (to Manny's work)

Two angles. Offense: for red-teaming open-weight releases, the question becomes whether USG-style gates survive adaptive attacks — e.g., out-of-distribution harmful styles, multi-step fine-tuning that slowly drifts hidden states out of the protected region, or orthogonal harmful tasks the gate wasn't calibrated for. Defense: if Manny ever ships or evaluates open-weight fine-tunes, this is the current best-practice release hardening to test against. It also matters for agent skills/models: fine-tune-to-backdoor (see yesterday's Claude Mythos 5 story) is the exact threat this defends against.

## What I Learned (plain English)

- You can block harmful fine-tuning by zeroing gradient directions in representation space — you don't need to control the whole training run.
- The calibrated protected region generalizes to nearby harmful samples, but the obvious attack is to find harmful samples *far* from the calibration set — distributional drift is the defense's weak spot.
- This is a race between calibration coverage (defender) and representation-space exploration (attacker) — the same dynamics as every other ML defense.
