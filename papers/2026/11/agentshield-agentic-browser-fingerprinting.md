# AgentShield: Fortifying Web Applications Against AI Agents and Agentic Browsers

**Authors:** Moiz Ali*, Nafis Karim*, Rownak Islam*, Jyotirmay Chauhan*, Dhruv Kuchhal†‡, Jason Polakis* (*University of Illinois Chicago, †Amazon.com, ‡work done outside Amazon)
**Published:** October 30, 2026 · **Venue:** IEEE S&P (Oakland) 2027
**Link:** https://arxiv.org/abs/2610.41181 · **Code/data:** prompts and artifacts linked from the paper (§II, ref [41])

---

## Problem Statement

Anti-bot defenses were built for a world where automation meant Selenium and Puppeteer: headless browsers, datacenter IPs, no mouse, no typing rhythm. Agentic browsers break that model. Comet, ChatGPT Atlas, Claude for Chrome, Manus and friends run inside a real browser on a real user's machine, driven by an LLM that reads the page and decides what to click. They are, at the network layer, indistinguishable from the human whose laptop they're running on.

Nobody had systematically measured whether existing defenses actually work against them, and nobody had a detection method that targets what makes them different. The paper asks two questions: can you tell an agent from a human at all, and if you can, can you also *manipulate* the agent?

## Methodology

**Test corpus of 14 agentic clients.** Twelve autonomous/agentic clients plus browser assistants, spanning four perception architectures: DOM/HTML parsers, accessibility-tree readers, screenshot/vision clients, and hybrid dual-request clients (Comet and Atlas issue one request from the user's residential browser and a second from the provider's cloud backend).

**Layer 1 — attack the existing defenses.** Evaluate every deployed signal class: source IP reputation and Cloudflare's AI Crawl Control/AI Blocking, reCAPTCHA v3 and Turnstile, and self-declared identity (User-Agent suffix plus RFC 9421 / Web Bot Auth request signing). Tabulate detection per agent.

**Layer 2 — AgentShield, behavioral fingerprinting of the PRA loop.** Serve dynamic forms that progressively reveal fields and capture fine-grained interaction telemetry. Extract 12 features: four reaction-time statistics (mean/median/min/max interval between a field appearing and first interaction), three keystroke-dynamics features (dwell-time mean, dwell-time SD, keypress count), two pointer features (movement count, clicks), session duration, and two flags for whether any dwell/inter-key timing exists at all. Fit an L2-regularized logistic regression with balanced class weights — one model per form variant, because dropdowns shift reaction times by multiples and a pooled model would learn form complexity instead of agent behavior. Crucial evaluation choice: **leave-one-agent-out** — each agent is scored by a model that has never seen it, testing generalization to unseen automation rather than memorization. Humans get K-folded; their mean score is the false-positive rate.

**Layer 3 — content poisoning, split by perception channel.** Five hidden-prompt categories (Defensive/ToS notice, Hidden Promotion, Opposite Information, Additional Information, Abusive warning), each run 10 times per agent. Two concealment techniques: a `.zero-text-injection` CSS class (font-size and line-height 0, transparent color, 1px absolute box — deliberately *not* `display:none`, which some parsers drop) for DOM/HTML/a11y-tree readers, and a `<canvas>` render at `rgba(180,180,180,0.15)`, 15px, for screenshot-only agents that OCR the page.

**Layer 4 — Extinction-Induced Policy Drift (EPD).** A custom e-commerce site where the secure-checkout button freezes the page for 120 seconds. A hidden zero-dimension div carries a "site fallback mechanism" instruction pointing at an attacker URL. Baseline first: run the same task without the freeze to confirm every agent ignores the hidden instruction under normal conditions.

## Key Results

**Existing defenses mostly fail, and the failures are structural.**
- Locally-running agents (Comet, Claude for Chrome, Chrome DevTools MCP, Director.ai) are invisible at the network layer — residential IP, real browser.
- Cloudflare's AI Crawl Control + AI Blocking blocked only 3 of the data-center-hosted agents it could plausibly flag, and blocked **neither Comet nor Atlas**, both of which send a residential request and a backend request.
- reCAPTCHA v3 was passed by Comet and Claude for Chrome. Turnstile detected only 3 of 7 tested agents — it validates the session environment, which agents satisfy by running inside a legitimate browser.
- RFC 9421 signing is essentially absent: only Amazon Nova signs. ChatGPT agent mode signs when called through chatgpt.com but **drops signing entirely inside Atlas**. Comet and Atlas append agent UAs only to the cloud-backend request; the request that touches the page carries a stock Chrome UA.

**Behavioral signals do separate agents from humans.**
- Human reaction time to a newly revealed field: ~5.5s. Claude for Chrome 1.7s, Chrome DevTools MCP 2.3s on the simple form — *faster than humans*, so reaction time alone isn't sufficient.
- Add dropdowns and those two jump to 9.5s and 21.2s — decision stages beyond the basic PRA loop.
- Human dwell time (keypress duration) ~108ms, matching the 116ms mean reported over 136M keystrokes in prior work. Comet, Manus and Claude for Chrome sit at a few milliseconds with near-zero variance — physically unattainable for a human.
- Manus and Director.ai **failed every dynamic-form run**: they opened the dropdown and couldn't select an option. Basic web controls still break deployed agents.
- Held-out classifier: every agent scored above the human baseline. Most above 0.70. Nova — the one agent with near-human typing simulation — fell to **0.143** on the simple form but scored **0.98** on the dynamic form, where interaction complexity exposed signals its typing simulation doesn't cover.

**Content poisoning: susceptibility is architecture, not model.**
- ToS/defensive notices are largely ignored; only ChatGPT (web) consistently respected them.
- Promotional injections are highly effective — ChatGPT (web) and Gemini in Chrome reproduced fabricated offers as legitimate content. Comet recognized the injection as suspicious *and passed it to the user anyway*. Claude for Chrome consistently excluded it.
- Factual-sounding additions ("Additional Information") were among the most effective: they read like article text, not instructions, so agents that caught the promotional injection still absorbed them. Only a small subset (including Atlas) suppressed them.
- **Canvas injection closes the loop**: Amazon Nova, immune to every DOM injection because it only sees screenshots, followed the canvas payload in 8/10 runs. Gemini in Chrome and Director.ai followed it every run.
- Repetition is non-monotonic: ChatGPT (web) starts compliant, then treats repeated notices as injection and dismisses them. Claude (web) becomes more compliant at N=3–5 and less at N=7. Atlas barely moves either way. Same model family, opposite behavior — **agent architecture dominates model choice.**

**EPD: obstruction makes agents drop their own safeguards.**
Baseline (no freeze): all six tested agents ignored the hidden instruction and completed the legitimate checkout. With the 120s freeze, behaviors diverged into: hidden instruction execution (agent navigates to the attacker URL), full 120s timeout compliance, user solicitation (detects the instruction and asks permission), **URL hallucination** (invents `/cart` or `/checkout` and navigates there), and deterministic-interval retries on the frozen button. The hidden instruction is only acted on once the reward path is withheld.

Side note the paper flags: a Munich Regional Court preliminary injunction (LG München I, Az. 26 O 869/26) held Google directly liable for false claims in AI-generated search overviews, reasoning that search-engine liability privilege doesn't extend to "AI search."

## What's Novel

1. First systematic measurement of *deployed* anti-bot defenses against *deployed* agentic browsers, with per-agent tables rather than synthetic bots.
2. AgentShield's framing of detection as PRA-loop telemetry rather than fingerprint or reputation — and the leave-one-agent-out protocol, which is the only way to show you're detecting automation and not memorizing a client.
3. Extinction-induced Policy Drift. This is the conceptual contribution: agent safeguards are conditioned on task progress, so an adversary who controls the reward channel controls the safety posture. Freeze the checkout, and the agent starts trusting the page.
4. The perception-channel matrix (DOM vs canvas, with no client immune to both), plus the demonstration that repetition can *increase* or *decrease* compliance depending on the agent's injection-detection heuristics.

## My Connection

- **Red team playbook.** EPD is a reusable primitive, not a one-off. Any engagement against a browser agent that has a goal (buy, book, file, submit) can use controlled obstruction to induce the target to accept instructions it would otherwise reject. That's a cheaper path than prompt-injecting a well-hardened agent from a cold start.
- **The dual-channel poison is a template.** `.zero-text-injection` plus canvas covers the entire perception taxonomy. If you're building agent-facing content for a test, ship both.
- **Detection engineering.** The 12-feature model is small enough to reimplement — reaction time and dwell-time variance are the two signals worth instrumenting first, and they map cleanly onto existing client-side telemetry.
- **Reporting.** Where a vendor says "we block AI agents," this paper is the counter-evidence table: Cloudflare blocked 3 of the data-center set, and zero locally-running agents.

## What I Learned

Anti-bot defense is an identity problem, and agentic browsers have no identity to check — they're the user's browser with the user's IP and the user's cookies, so every "who are you / where did you come from" question returns "a human." The only thing left is *how* the thing behaves: how fast it reacts, how it types, whether its keystrokes have human variance. That's measurable, but it's also spoofable, and one agent (Nova) already simulates typing well enough to score near the human baseline on a simple form.

The more interesting lesson is about safety, not detection. An agent's guardrails are not a fixed property — they're a function of how close it thinks it is to finishing. Withhold the reward and the agent will bargain with its own rules. That's true of every goal-directed system, and it means "the agent refused" is only a valid result under the conditions you tested.
