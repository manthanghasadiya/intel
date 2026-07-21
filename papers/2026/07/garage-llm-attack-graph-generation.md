# GARAGE: Characterizing the Automation Boundary in LLM-based Attack Graph Generation

- **Link:** [https://arxiv.org/abs/2607.18108](https://arxiv.org/abs/2607.18108)
- **Authors:** Daekwon Pi, Sangho Lee, Young Hun Lee, Huy Kang Kim
- **Institution:** Korea University (Graduate School of Information Security)
- **Published:** July 20, 2026
- **Code/Dataset:** Not explicitly mentioned in abstract

## Problem Statement (Plain English)

Current automotive security relies on Cyber Threat Intelligence (CTI) — vulnerability reports, incident docs, and threat advisories. But these are unstructured, noisy, and domain-specific. Security analysts manually piece them together to understand attack paths. GARAGE asks: can an LLM + RAG pipeline automate the construction of attack graphs for vehicle architectures, and where does it hit the limits of reliability?

## Methodology (Technical)

1. **Knowledge Base Construction:** Synthesizes 12,786 CVEs and 140 incident reports into a STIX 2.1 and Auto-ISAC ATM-compliant knowledge base using RAG (Retrieval-Augmented Generation).
2. **Attack Graph Generation:** Formalizes tactical-pattern-level scenarios through granular kill chain analysis. Uses Leave-One-Out experiments (320 total) to test whether the framework can transfer security knowledge to entirely unseen vehicle architectures.
3. **Cost-Performance Analysis:** Evaluates across various LLM tiers to guide practical deployment as a TARA (Threat Analysis and Risk Assessment) support tool within human-in-the-loop workflows.

## Key Results

- The framework can accurately transfer security knowledge to unseen vehicle architectures.
- Positioned as scalable TARA support — not fully autonomous, but reducing analyst workload significantly.
- Cost-performance analysis provides deployment guidance across LLM tiers (cheaper models for routine cases, frontier models for edge cases).

## What's Novel

- First systematic treatment of LLM-generated attack graphs for **automotive** security, not general IT.
- Integration of RAG with STIX 2.1 and Auto-ISAC compliance (regulatory-aligned).
- Large-scale Leave-One-Out validation (320 experiments) that tests generalization to unseen architectures.
- Explicitly characterizes the "automation boundary" — what the LLM can reliably do vs. what needs human review.

## My Connection (to Manny's Work)

- Attack graph generation is directly relevant to mcpsec's red-teaming methodology — the same RAG+LLM pipeline could generate attack trees for MCP deployments.
- The STIX 2.1 alignment means the knowledge base format is interoperable with existing threat intelligence tooling.
- The cost-performance analysis methodology (which LLM tier for which task) is directly applicable to mcpsec's agent orchestration decisions.
- The "automation boundary" concept is exactly what mcpsec needs: where can an agent autonomously handle security tasks vs. where must a human sign off.

## What I Learned (Plain English)

Attack graph generation for automotive systems is a domain-specific problem that standard LLMs can't solve without structured knowledge retrieval. By combining RAG with regulatory-compliant knowledge formats (STIX 2.1), GARAGE shows that LLMs can produce useful attack graphs for novel vehicle architectures — but need human oversight for critical edges. The 320 Leave-One-Out experiments are a strong validation methodology: test generalization by removing one architecture, training on the rest, and measuring how well the model handles the unseen case. This is a template for how to validate any AI security tool.
