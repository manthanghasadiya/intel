# LoginTrap: Uncovering Task-Agnostic Phishing-Style Indirect Prompt Injection Attacks against LLM-based Web Agents

- **Link:** https://arxiv.org/abs/2608.04741
- **Authors:** Longtao Guo, Zelin Zhang, Kaifeng Huang, Yang Shi
- **Institution:** Not stated on arXiv abstract page
- **Published:** 2026-08-05
- **Code:** Not stated

---

## Problem Statement (plain English)

Web agents log into real services on behalf of users — so the login step is a sensitive boundary holding credentials and session tokens. Prior work showed malicious webpages can manipulate agent actions, but nobody had shown an attacker can *force a login* and then steal the credentials/session end-to-end. LoginTrap asks: can a black-box attacker make an agent voluntarily log in to an attacker-controlled page, no matter what task the user gave it?

## Methodology (technical)

- **Threat model:** black-box attacker who controls webpage context and the induced login flow; does NOT know the user's task or the agent's internals.
- **Attack construction:** a fuzzing-inspired process generates page-specific indirect injections from the webpage context, so each injection looks naturally embedded in the page. The payload frames login as a plausible prerequisite for "continuing the task" (e.g., "session expired, please sign in to continue"), steering the agent to a controlled login page.
- **Goal:** end-to-end — the agent authenticates against the attacker's page, leaking credentials or attacker-usable session state.
- **Evaluation:** across multiple LLM backbones, agent architectures, and defense configurations, measuring end-to-end attack success (agent actually performs login on the malicious page).

## Key Results (with numbers)

- **86% average end-to-end attack success** across LLM backbones.
- Remains effective across agent architectures and existing defenses — i.e., current defenses don't stop login inducement.
- Task-agnostic: works regardless of what the user's actual task is.

## What's Novel

- First systematic treatment of *login inducement* as an attack primitive for web agents — not just action manipulation, but crossing the authentication boundary.
- Identifies a structural gap: authentication-aware defenses for agent pipelines don't exist yet.
- Black-box and task-agnostic, which makes it broadly applicable and hard to patch with task-specific rules.

## My Connection (to Manny's work)

This is a must-know attack class for any agent red team: browser-use style agents, MCP tools that hold OAuth sessions, and internal agent harnesses with SSO. The practical check for Manny: does the agent under test ever navigate to a login page *it wasn't instructed to visit*, and does it enter credentials there? A cheap detection hook is to monitor for navigation to unexpected auth domains. Also pairs with the CoreBreak story — both exploit trust in the environment rather than the model.

## What I Learned (plain English)

- The login boundary is the soft underbelly of web agents — attackers don't need to break the model, just make login look routine.
- "Session expired, sign in to continue" is apparently a near-universally trusted framing across agent backbones — a fantastic social-engineering vector for machines.
- Defenses that focus on injection text won't stop this; you need authentication-flow guards (allowlisted auth domains, credential-handling rules that never autofill on unexpected origins).
