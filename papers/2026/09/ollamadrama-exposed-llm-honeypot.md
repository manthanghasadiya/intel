# OllamaDrama: Designing and Deploying a Honeypot to Measure Attacks on Exposed LLM Infrastructure

**Authors:** Karina Elzer, Niklas Netterstrøm Johansen, Emmanouil Vasilomanolakis
**Institution:** Not stated on the arXiv abstract page (deployments spanned cloud and university networks; verify affiliations in the PDF before citing)
**Published:** September 24, 2026
**Link:** https://arxiv.org/abs/2609.29757
**Code:** Not linked on the abstract page — the honeypot is called **Ollure**
**Subjects:** cs.CR (primary)

---

## Problem Statement (plain English)

Self-hosted LLM endpoints are being stood up faster than anyone is securing them. `ollama serve` exposed to the internet is now as common as an open Redis or Elasticsearch port was a decade ago — except nobody has measured what actually happens to them. Are these hosts just quietly mined, or are attackers doing LLM-aware things like model theft, prompt injection, and tool abuse? This paper answers with real data: **what attacks do publicly exposed LLM services actually receive?**

## Methodology (technical)

- **Ollure** — a honeypot that emulates the **Ollama API** at low/medium interaction, with **no backend LLM**. It presents the interfaces attackers expect (model listing, pulling, generation endpoints) and truthfully captures requests rather than fabricating model output.
- Deployed across **four environments** — a mix of cloud and university networks — for **84 days**.
- All inbound interactions logged and classified into an attack taxonomy spanning two layers:
  - **Infrastructure layer** — the host/container/network around the service.
  - **LLM layer** — the model API and prompting interface itself.
- Traffic segmented into automated discovery/fingerprinting vs. targeted exploitation.

## Key Results (with numbers)

- **290,887 interactions** from **2,793 unique source IP addresses** across **84 days**.
- The bulk of traffic was **automated discovery, fingerprinting, and model enumeration** — i.e. mass scanning, not targeted attacks.
- Real exploitation was also observed, in both layers:
  - **Infrastructure**: path traversal and SSRF probes; **RCE payloads**; **cryptocurrency-mining payloads**; resource-exhaustion attempts.
  - **Model management abuse**: unauthorised model pull/delete/management operations.
  - **LLM layer**: **prompt injection**, information extraction (smuggling secrets/system prompts out), and **agent-oriented tool use**.
- Practical takeaway: an exposed Ollama host is misused for both host compromise *and* free compute / inference abuse, and the boundary between "someone port-scanned you" and "someone got RCE" is thin.

## What's Novel

There is abundant survey/advisory material about the *theoretical* risks of exposed LLM infra, and a lot of "X% of servers found exposed" port-scan journalism. This is one of the first pieces of **longitudinal empirical honeypot data** that shows what requests actually arrive — and it separates host-level exploitation from **LLM-layer** techniques like prompt injection and information extraction arriving unsolicited from the internet. The "no backend LLM" design is also a nice methodological trick: it lets you watch attacker expectations and probes without the ethical and cost problems of running a live model for strangers.

## My Connection

Directly relevant to any self-hosted or edge-deployed agent stack. Manny's world increasingly runs local models (Ollama/LM Studio/vLLM) as *tool backends* for agents — and this paper shows those endpoints attract prompt-injection and info-extraction traffic from the open internet, not just port-scanning and cryptominers. Pair it with today's **x47.c** botnet story (AI API draining) and the picture is joined up: exposed LLM services are now a two-sided target — you lose the host *and* the keys/compute. The taxonomy here is a ready-made checklist for any "don't expose your inference server like 2015 Redis" piece.

## What I Learned (plain English)

If you put an LLM server on the internet with no auth, people will find it in hours and they will not just mine crypto on it — they will try to talk your model into leaking its secrets and they will try your own tool endpoints against you. The scan is automated, but the prompt injection is deliberate. "It's just my local test box" is not a security model; the internet does not distinguish between your laptop and your prod cluster.
