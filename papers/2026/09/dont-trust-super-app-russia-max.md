# Don't Trust the Super-App: A Case Study of Russia's Max

**Authors:** Richa Priyanka, Aaron Ortwein, Joel Reardon, Michael Specter, Piyush Kumar Sharma, Roya Ensafi
**Institution:** University of Michigan (Ensafi / Censored Planet group) and University of Calgary (Reardon) — standard for this authorship set; confirm exact affiliations in the PDF.
**Date:** September 10, 2026
**Link:** https://arxiv.org/abs/2609.11814
**Code:** Not stated in abstract

---

## Problem Statement (plain English)

"Super-apps" (WeChat, Gojek, and now Russia's MAX and Iran's Bale) host third-party *mini-apps* inside one big app. A decade of security research has quietly assumed the super-app itself is a **trusted intermediary** — that it won't abuse the mini-apps or their users. The authors argue that assumption is unjustifiable, and demonstrate that a malicious super-app can silently undermine the security and privacy of both mini-apps and users **without leaving any trace**.

## Methodology (technical)

- **Threat-model inversion:** instead of assuming a trusted host, the authors enumerate what a *hostile* super-app can do from its architectural position.
- **Capability demonstration on MAX** (Russia's state-linked super-app): concrete, working demonstrations of each privileged capability available to the host.
- **Corroborating context on other ecosystems:** WeChat (passive cross-mini-app activity tracking at scale), MAX's parent company (entanglement with state prosecution of online speech), and Iran's Bale (functioning through the world's longest internet shutdown via state backing) — used to argue the trust assumption is not just theoretically weak but empirically contradicted.
- **Architectural-general argument:** because the capabilities follow from the privileges the super-app model *grants by design*, the findings apply to **any** super-app, not just MAX.

## Key Results

A malicious super-app host can, with no user-visible trace:

- **Capture mini-app UI** (read what the user sees inside a mini-app).
- **Read and write mini-app local storage** (access and tamper with mini-app state/secrets).
- **Inject arbitrary JavaScript** into a mini-app's runtime.
- **Mediate mini-app network traffic** (intercept/alter communication).
- **Control authentication context**, enabling **silent user impersonation**.
- These capabilities are **architectural** — they "manifest themselves in any super-app because of the architectural privileges granted to them by design." The paper's position is that mobile-OS and app-store interventions are "urgently needed to close this architectural blind spot."

## What's Novel

- **Breaks the trust assumption** that has underpinned super-app security research for a decade — prior work studied *mini-app* risk; this studies *host* risk.
- **A complete, demonstrated capability set** (UI capture, storage R/W, JS injection, traffic mediation, auth-context control) rather than isolated findings.
- An **architectural** result: the vulnerability class is a property of the model, not of one country's app — MAX is the case study, not the exception.
- Survives the usual "but that's a state actor" dismissal by showing no special access is required — the privileges are baked in.

## My Connection

This is the client-side twin of Manny's agent/plugin-security work. The super-app-host threat model — a privileged host that can read UI, rewrite runtime, mediate network, and spoof identity with no trace — is structurally identical to the **MCP host / agent-runtime** model: a privileged orchestrator that reads tool descriptions, injects instructions, and controls credentials. Both are "the platform is the attack surface, and the platform is implicitly trusted." Worth mining for a comparison piece: "Mini-apps and MCP servers have the same trust problem — the host is unaccountable." The "no trace left" property is the killer detail for detection engineering: if the host can act invisibly, log integrity can't be assumed.

## What I Learned (plain English)

When you let one app run all your other apps, that first app can see and change everything — read your screens, rewrite their code, intercept their traffic, and even log in as you — and you'd never know. We built this assumption of a "good" super-app into a decade of research. The paper's point is simple and hard to argue with: the trust was never justified, and the fix has to come from Android/iOS and the app stores, because the app itself has no incentive to give up those powers.
