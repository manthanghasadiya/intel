# Lilith: Backdoor Generalization under Training-Inference Trigger Shift

- **Authors:** Zhou Feng, Jiahao Chen, Chunyi Zhou, Yuan Su, Tianyu Du, Yuwen Pu, Jianhai Chen, Jinbao Li, Shouling Ji
- **Link:** https://arxiv.org/abs/2607.26099
- **Published:** July 30, 2026 (arXiv:2607.26099)
- **Code:** Not specified
- **Subjects:** cs.CR, cs.LG

---

## Problem Statement

Existing backdoor attack research assumes the trigger used at inference time is the same as, or a simple variation of, the trigger embedded during training. This leaves a critical blind spot: can a backdoor learned from one trigger generalize to an entirely different trigger family never seen during training? If so, current backdoor defenses (which test only exact or near-exact trigger reuse) dramatically underestimate real attack surface.

## Methodology

Lilith is a black-box "anchor-to-family" framework:

1. **Anchor Injection:** Using only disjoint surrogate data, Lilith induces a compact vulnerability in the target model with a single training-time anchor trigger.
2. **Family Construction:** At inference time (training not required), Lilith constructs a bounded family of triggers that preserves the anchor-induced representation geometry.
3. **Mathematical Characterization:** The paper introduces "anchor clearance" and "family reach" — sufficient conditions for family-wise target preservation under local regularity and bounded surrogate-victim discrepancy.

The approach is black-box: the attacker does not need access to the victim model's architecture, weights, or training data.

## Key Results

- High family-wise attack success across multiple datasets (CIFAR-10, ImageNet subsets, etc.)
- Multiple architectures tested (ResNet, VGG, ViT)
- Various poisoning rates — Lilith maintains efficacy even at low poisoning rates
- Survives existing backdoor defenses designed for exact-trigger detection
- Limited utility degradation on benign inputs
- Small trigger generalization gap

The core finding: family activation depends on representation alignment, not the trigger proposal mechanism — meaning defenses that only check for exact trigger patterns miss the broader threat.

## What's Novel

- **First systematic formulation** of backdoor generalization under training-inference trigger shift
- Proves that a single anchor trigger can generalize to an entire family of unseen triggers at inference time
- The "anchor clearance" and "family reach" concepts provide formal characterization
- Demonstrates that existing defenses have a fundamental blind spot — they test for what they know, not what can be generated

## My Connection

For Manny's red team work, this is directly relevant to evaluating ML supply chain security. If an organization uses a third-party model or training pipeline, a single poisoned sample can enable a whole family of inference-time attacks. This expands the threat model from "detect the known trigger" to "any trigger in a learned family could activate the backdoor." Also relevant for evaluating how agent systems that use fine-tuned models might be subverted.

## What I Learned

Backdoor attacks are more dangerous than standard evaluation suggests. The ability to generalize from one training trigger to an unlimited inference family means backdoor detection must be representation-based, not pattern-based. This is analogous to the shift from signature-based antivirus to behavioral detection — the same lesson, now in ML security.
