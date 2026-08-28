# The Framing Gap: Indirect Prompt-Injection Exfiltration Defeats Surface-Level Defenses in Tool-Using Agents

- **Link:** https://arxiv.org/abs/2608.27092
- **Authors:** Md Habibur Rahman, Jaeho Kim
- **Institution:** Not listed in arXiv metadata
- **Published:** 2026-08-27
- **Code:** Not listed

## Problem Statement (plain English)

A tool-using agent that browses attacker-controlled web content while holding a secret (API key, session token, private data) is exposed to indirect prompt injection: the page's content can instruct the agent to leak the secret. Many defenses try to make the *model* recognize malicious instructions. This paper shows recognition is the wrong layer: the exact same leak request, reframed to look like a legitimate mechanism (a mandatory integrity signature, a config field, a "trusted" host), goes from being refused 100% of the time to succeeding 100% of the time. The gap between "obvious injection" and "plausible-looking instruction" is the *framing gap*.

## Methodology (technical)

- Safe synthetic lab: canary secret, mock tools, matched clean-vs-poisoned pages so refusal and success are measured against identical baselines.
- Six models tested; ten overt injection classes as the control; then the identical leak reframed three ways (integrity signature, config field, look-alike trusted host).
- Attack-cost analysis at three levels: paraphrase a known mechanism (3 wordings), swap the field inside a known-effective template, author a fresh page with a new mechanism.
- Defense ablations: confidentiality-policy removal, destination allow-list, planner/reader capability split, an "in any form" policy clause, SecAlign fine-tuning (CCS 2025), channel separation, output-normalizing guard (tested with ROT13 held-out encoding).

## Key Results (with numbers)

- **The gap:** overt injections refused (gpt-4o 0%); reframed leaks succeed 0% → **100%** on gpt-4o.
- **Attack cost is three-level:** paraphrasing a known mechanism is trivial (**96% at just 3 wordings**); swapping a field inside a known-good template is cheap (**up to 60%**); authoring a fresh page around a *new* mechanism is hard (**0/130**) — so the reusable attacker asset is the *template*, not the mechanism.
- **Mechanism diagnosis:** it's instruction/data confusion, not defeated alignment — removing the confidentiality policy leaves base attacks at 0% and only moves reframing 31.9% → 38.1%.
- **What closes the gap (payload-blind only):** destination allow-list with closed destinations (**0%**) and a capability-isolating planner/reader split (**0%**).
- **What fails:** "in any form" policy clause closes it at the acting model (**0%**) but is brittle — dropping the catch-all reopens it to **48.8%**; SecAlign fine-tuning on a tool agent leaves **32.5%**; channel separation **38.8%**; an output-normalizing guard loses to a held-out encoding (ROT13 → **100%**).

## What's Novel

- Rigorous demonstration that surface-level, content-based defenses are structurally insufficient: the robust controls are *payload-blind* (constrain the destination, isolate the capability) rather than any policy the acting model applies to the payload itself.

## My Connection (to Manny's work)

Direct blueprint for red-team exfil tests and for hardening agent deployments: stop relying on the model recognizing "malicious" content — put egress allow-lists and planner/reader isolation in front of anything that holds secrets. Also a warning about eval hygiene: a defense that looks great on obvious injections will collapse under reframing; test with held-out encodings and template-swapped payloads.

## What I Learned (plain English)

- If a leak is framed as a routine mechanism (signature, config field, trusted host), models will happily perform it — recognition-based defenses are fighting the wrong battle.
- Attackers stockpile *templates*, not tricks: a known-effective template is a reusable asset that survives paraphrasing and model changes.
- The only defenses that held were ones that didn't look at content at all: destination allow-lists and capability isolation. If the agent physically cannot reach the exfil destination, the framing doesn't matter.
