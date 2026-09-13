# From Specs to Apps: Verifying and Monitoring Models of Signal and WhatsApp

**Authors:** Moustafa Said, Aurora Naska, Kevin Morio, Robert Künnemann
**Institution:** CISPA Helmholtz Center for Information Security (Saarbrücken)
**Published:** Sep 10, 2026 | **Venue:** ACM CCS '26 (full version)
**Link:** https://arxiv.org/abs/2609.11882 | **Code/Artifacts:** instrumentation + SpecMon-based monitoring described in paper

---

## Problem Statement (plain English)

The Signal protocol is the security backbone for billions of messaging users — it powers WhatsApp and the Signal app. Cryptographers have proven strong safety properties about the *protocol specification*, but almost nobody has proven that the *running implementations* actually behave like the spec. That gap matters: a protocol can be provably secure on paper while the app on your phone has drifted, misconfigured, or been forked in ways that quietly weaken it. The paper closes that gap for two of the most consequential deployments: WhatsApp Web and Signal Desktop.

## Methodology (technical)

- **Runtime monitoring with SpecMon:** The authors apply SpecMon, a recently proposed runtime monitor, to check whether *observed executions* conform to a formal protocol model.
- **Instrumentation:** They instrument WhatsApp Web and Signal Desktop to capture their interactions with both the network layer and the cryptographic components — yielding a trusted event stream to feed the monitor.
- **Formal models:** They build two multiset-rewrite models compatible with **Tamarin** (the symbolic protocol verifier): the first model of WhatsApp Web's implementation of the Signal protocol, and the most detailed model to date of Signal's original protocol.
- **Verification + monitoring, combined:** On the core components they *verify* authentication and secrecy properties symbolically; separately they *monitor* real executions for conformance to those models relative to trusted event extraction and a symbolic (Dolev-Yao-style) abstraction.
- **Fault injection:** They demonstrate the monitor detects deliberately injected security faults, and they add fuzzing to shake out edge cases. Reproducibility is explicitly evaluated.

## Key Results (with numbers)

- Monitoring **established that observed executions conform** to the derived models (relative to the stated abstraction assumptions).
- **Authentication and secrecy verified** for the core components of the Signal protocol.
- Monitoring revealed **previously undocumented differences between the upstream libsignal library and WhatsApp's fork** — i.e., real divergence that spec-only analysis would never surface.
- The monitor detects **deliberately injected security faults** with **low overhead** in the measured setting.
- Effort: building the WhatsApp Web model, instrumenting the app, adding fuzzing, and running experiments took **three person-weeks**.

## What's Novel

1. **First model of WhatsApp Web's actual Signal-protocol implementation** (as opposed to the spec), and the most detailed model yet of Signal's original protocol.
2. **Runtime conformance checking as the bridge** between symbolic verification and shipping code — using a real monitor (SpecMon) on real apps rather than toy clients.
3. **Empirically demonstrated fork divergence** — concrete evidence that "same protocol, different implementation" produces measurable security-relevant differences.

## My Connection (to Manny's work)

This is the strongest recent template for **verifiable behavior of a deployed system against a formal model**. Agent harnesses and MCP servers are in exactly the pre-CSS situation Signal was in a decade ago: rich informal reasoning about what the system "should" do, thin evidence about what the running binary actually does. The move worth stealing is the *dual artifact* — a symbolic model you can verify against, plus a runtime monitor that flags drift from that model in production. If you're building agent-assurance tooling, "spec model + conformance monitor + injected-fault regression suite" is a shippable product shape, not a research fantasy, and this paper shows the whole loop in three person-weeks.

## What I Learned (plain English)

You can check a real messaging app against a formal model of its protocol *while it runs*, cheaply enough to be practical, and doing so finds real differences between a library and its fork that no amount of spec-reading would reveal. Verification isn't only for idealized protocols anymore — it can babysit the code people actually ship.
