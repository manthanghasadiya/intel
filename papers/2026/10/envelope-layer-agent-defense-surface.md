# The Envelope Layer: An Underestimated Surface of LLM Agent Defense

**Authors:** Yuelin Han | **Affiliation:** not listed on the arXiv record
**Published:** October 27, 2026 | **arXiv:** [2610.37415](https://arxiv.org/abs/2610.37415) (cs.CR)
**Code:** not released

---

## Problem Statement

Agent frameworks wrap inbound content in an envelope — a header, a sender label, a role tag, a delimiter, a marker saying "the following is untrusted tool output." Practitioners treat this as hygiene. The assumption behind it is that a strong label ("⚠️ UNTRUSTED EXTERNAL CONTENT") makes the model more likely to treat what follows as data rather than instruction.

Prior work established the envelope layer as a defense surface distinct from the model layer and the agent/scaffold layer, but tested it briefly. This paper asks the four questions that brief test skipped: does label *strength* actually matter, does it matter when the envelope itself can be forged, do different frontends/scaffolds behave differently, and — the one nobody tests — how does existing agent memory interact with all of it?

## Methodology

**Test vehicle.** A2A-TIBA, an agent-to-agent trust-inheritance attack. Content enters through one agent and is relayed to another, which is precisely where envelope labeling is supposed to do its work.

**Framework.** Follows the GDA Measurement methodology and the A/B/C/D outcome taxonomy from the same line of work. Primary metric is **semantic breach rate** (did the agent act on the injected instruction), with **semantic refusal rate** as the complementary reference — so the paper distinguishes "the model refused" from "the model never engaged," which most injection papers conflate.

**Design.** A two-by-two factorial experiment crossing **label strength** (strong vs weak, against an unlabeled baseline) with **envelope authenticity** (authentic vs forgeable), run across multiple frontends. Then a separate **memory experiment** crossing success memory and failure memory — i.e. the agent has previously succeeded or previously been refused on similar tasks.

**Why the factorial matters.** It separates two things that get bundled: the label's *semantic content* and the label's *trustworthiness as a signal*. If a strong label only helps when the envelope can't be forged, then the label was never doing the work — the authenticity was.

## Key Results

| Condition | Effect on semantic breach rate |
|---|---|
| Authentic envelope, strong label | **consistent reduction** |
| Authentic envelope, weak label | not necessarily better than unlabeled baseline |
| Forgeable envelope, strong label | benefit **weakened**, direction varies by frontend |
| Success memory present | breach rate **rises** |
| Failure memory present | breach rate **falls** |
| Both memories present | returns to **no-memory baseline** |

- **Strong labels work — but only under authenticity.** With a real envelope, strong labeling consistently lowers breach rate. Weak labeling is not reliably better than no label at all, which means the middle of the design space is a dead zone: if you're going to label, label hard.
- **Forgery collapses the benefit.** When the attacker can forge the envelope, the strong-label advantage weakens, and *the direction of the effect varies across frontends* — in some scaffolds a strong label becomes a liability. This is the paper's most operationally important finding: envelope defense is only as good as the guarantee that the attacker can't write the envelope.
- **Memory dominates labels.** No label manipulation in the study moves the breach rate as much as a single prior success or failure. Success memory raises breach rate; failure memory lowers it; when both are present, most combinations fall back to baseline — the agent's behavior is being set by whichever experience is easier to retrieve, not by the trust annotation in front of it.
- **The envelope is two surfaces, not one.** The paper refines the layer into an **admission channel** (does this content get in) plus a **context assembly surface** (how it's presented, ordered, and framed once assembled) — and argues the security boundary is a dynamic outcome of label claims, frontend permissions and history rather than a fixed property of the label.

## What's Novel

1. **Falsifying the "strong label = safer" heuristic.** Showing that weak labels can be worse than nothing, and that strong labels invert across frontends, turns a folk practice into a measured one with a failure mode.
2. **Authenticity as the actual security property.** The paper's real claim is that envelope defense is a *provenance* defense wearing a formatting costume. If the envelope isn't unforgeable, you're decorating the attack.
3. **Memory as a confounder nobody controls for.** Prior agent-injection work almost universally runs a fresh session. This paper shows prior success experience is a larger driver of breach than the defense being evaluated — which means a chunk of published defense efficacy may be an artifact of amnesiac agents.
4. **Splitting admission from assembly.** Recognizing that "did we let it in" and "how did we lay it out" are separable controls with separable failure modes.

## My Connection

This is a direct argument against the way most MCP harnesses implement tool-output trust. The typical pattern is a text wrapper injected into the context: `[TOOL RESULT from server X] ... [/TOOL RESULT]`. That wrapper is generated by the same process that concatenates attacker-controlled strings, so it is forgeable by construction — a malicious server returning a string containing a fake closing tag plus a fake system header has written the envelope. Per this paper, that's the condition under which strong labels stop helping.

The actionable version: envelope integrity has to be enforced *outside* the string domain — structured content blocks with provenance carried in the message schema (not in text), and rendering that cannot be spoofed by content. That's an MCP protocol-level fix, not a prompt-level one.

The memory finding is the more disturbing one for red-teaming. If success memory raises breach rate, then any agent that has ever been successfully injected once — including in a test, including in a different tenant, including via a memory store shared across sessions — is a more exploitable agent. Persistent memory turns a one-time win into a standing modifier, which reframes memory poisoning from "a way to plant a payload" to "a way to shift the base rate for every future attack."

## What I Learned

The framing I'd been carrying was wrong in a specific way: I thought of the envelope as a *signal to the model* ("this is data, be careful"). It's actually a *claim about provenance*, and its security value is entirely a function of whether the claim can be verified. A signal can be strengthened by making it louder. A provenance claim can only be strengthened by making it unforgeable — and no amount of uppercase, emoji or repetition substitutes for that.

The second lesson is about experimental design in agent security. Any evaluation run in a fresh context is measuring a system that does not exist in production. Agents have history, and the paper shows history outweighs the defense under test. Before believing any defense number, the first question should be: what did this agent remember when it was measured?
