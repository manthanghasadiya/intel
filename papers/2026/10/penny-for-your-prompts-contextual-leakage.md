# Penny for Your Prompts: A Measurement Study of Contextual Leakage in AI Chat Platforms

**Authors:** Paschalis Bekos, Billy Tsouvalas, Nick Nikiforakis, Michalis Polychronakis
**Published:** October 27, 2026
**Link:** https://arxiv.org/abs/2610.37467
**Code:** Not stated in the listing

---

## Problem Statement

You type something into a chatbot that you would never type into a search box. People disclose medical symptoms, legal problems, employment disputes, and relationship details to AI chat interfaces under a reasonable expectation that the conversation is between them and the model. But these platforms are web applications, and web applications carry analytics stacks: session replay, advertising pixels, third-party tag managers. Those scripts run *alongside* the chat widget, and nobody has systematically measured what they collect.

The gap is measurement, not speculation. Existing privacy research covers web trackers generically, and covers LLM privacy in terms of training data and memorization. Neither answers the question a user actually has: when I paste a prompt into this specific product, what leaves the page, and can it be tied to me?

## Methodology

**Scale.** The first systematic large-scale measurement study of contextual leakage in AI chat interfaces — 42 platforms, spanning:
- General-purpose assistants
- Domain-specific assistants in health, legal, and education

**The instrumentation is the contribution.** Network interception alone is what everyone has done before, and it's blind to *why* a request fired. The authors instead instrument the browser at the JavaScript engine level, which buys two things:

1. **Action attribution** — each outbound data flow is bound to the exact user action that triggered it (typing, sending, switching chat, opening a sidebar). This distinguishes "the platform leaks your prompt when you send it" from "the platform leaks a session identifier on page load."
2. **DOM capture attribution** — for each flow, *which* DOM capture mechanism retrieved the content. This matters because it separates reading an input field's value from reading rendered message text from reading a `data-` attribute, and those have different fix surfaces.

That granularity is invisible at the network layer, where all three collapse into "a POST went to a tracker."

**Threat model.** Third-party analytics, advertising, and session-replay scripts operating silently alongside the chatbot. No platform compromise required, no vulnerability, no attacker — just the ordinary commercial web stack doing what it does, pointed at the most sensitive text a user ever types.

## Key Results

| Finding | Count (of 42 platforms) |
|---|---|
| Leak conversation titles to third-party trackers | **21** |
| Leak **verbatim** user prompts | **9** |
| Send user email addresses to third-party trackers | "substantial amount" (not precisely counted in the abstract) |

The two findings compound. Conversation titles alone are already disclosure — auto-generated titles are distilled summaries of the conversation, so "Chest pain after starting lisinopril" leaks the payload without the body. Verbatim prompt leakage is strictly worse.

The email finding is the one that changes the legal posture. A leaked email address turns an anonymous behavioral record into a named one. Once a tracker holds *both* the conversation title and the account's email, the conversational context is associated with a real identity — which is the difference between "some browser leaked a string" and "this named person discussed this medical condition."

## What's Novel

1. **First measurement at this scale for AI chat specifically.** 42 platforms, three verticals. Prior work measured generic web tracking or LLM memorization; nobody had pointed the instrumentation at the chat surface itself.

2. **JS-engine-level attribution with per-action granularity.** This is a methodological advance reusable beyond this paper. Being able to say "the leak fires on message send, via the input field's value" instead of "a tracker received data" is what makes a finding actionable for a vendor.

3. **Vertical stratification.** Health, legal, and education assistants are where the disclosures are most damaging and where the regulatory exposure is highest. Measuring them separately from general-purpose assistants is the right design.

4. **Reframing web privacy issues in an AI context.** The authors' framing is that these are *traditional* web privacy bugs, not novel AI vulnerabilities — which is a useful corrective to a field that keeps looking for exotic prompt-injection attacks while the analytics tag is right there.

## My Connection

- **Directly relevant to client assessments of any AI chat product.** "Do you leak prompts to third parties?" is now a concrete, measurable question with a known base rate: roughly half the platforms tested leak titles, roughly a fifth leak prompts. That's a finding you can predict before you test.
- **The instrumentation technique is reusable.** JS-engine-level attribution bound to user actions is exactly how you'd audit an agent UI for what it ships to telemetry — and agent platforms ship far more context than chat platforms do.
- **It's a clean example of the difference between a vulnerability and a design choice.** Nothing here is a CVE. Every one of these flows is a commercial analytics integration working as configured. That's the harder conversation with a client, and this paper gives you the numbers to have it.
- **Connects to the data-handling reality of agent logs.** This week's Warp breach exposed agent decision logs; this paper shows the same class of data leaving through the front door by design.

## What I Learned

The most sensitive text on the internet is now typed into textareas that sit inside pages running advertising SDKs, and the industry has not adjusted. We spent a decade teaching people "don't put your SSN in a web form" and then built a product category whose entire value proposition is "tell me anything" — on top of the same tag-manager stack.

The operator lesson is about where to look. When you scope an AI product assessment, the JavaScript is not background noise, it's an exfiltration channel that requires no attacker and generates no alert, because it's the intended behavior. And the defensive recommendation isn't "detect the leak" — it's "audit the tag inventory with the same seriousness you audit the model," because the tracker was there before the chatbot and will be there after.
