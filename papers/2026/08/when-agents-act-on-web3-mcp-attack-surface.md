# When Agents Act on Web3: An Attack-Surface Survey of MCP, Skills, and Tool Calling

- **Paper:** https://arxiv.org/abs/2608.17275
- **Authors:** Rabimba Karanjai, Yang Lu, Nour Diallo, Wujie Xiong, Lei Xu, Weidong (Larry) Shi
- **Published:** Submitted Aug 18, 2026 (cs.CR)

## Problem Statement (plain English)

AI agents used to read the web; now they act on it — signing transactions, moving funds, mutating state. When those agents operate on public blockchains through MCP servers, skills, and tool calling, an attack isn't a recoverable mistake: blockchain transactions are irreversible, and a compromised agent can drain or lock value permanently. This survey maps the full attack surface of agents acting on Web3 and asks how well existing defenses hold up.

## Methodology (technical)

- Survey + taxonomy: organizes the fragmented MCP-security literature into an attack-surface taxonomy for agent/Web3 interactions.
- Empirical observation: across the MCP ecosystem, the share of deployed tools that **modify external state** has risen from **27% to 65%** of tool use.
- Argues four blockchain-layer properties qualitatively change the agent threat model: **irreversibility, signing authority, continuous autonomy, sequence-level composition**.
- Contributes a **Web3 risk-mapping matrix**: each attack class → amplified impact, responsible amplifiers, representative mitigation, residual gap.
- Evaluates existing defenses (blockchain-based mechanisms, model-level safety) against the matrix.

## Key Results (with numbers)

- Measured protections stop **fewer than 30%** of attacks.
- Model-level safety refusals stop **fewer than 3%**.
- Tool-state mutation share in MCP: **27% → 65%** — agents are increasingly "doers," making the Web3 surface structurally larger.
- Defenses are "improving but insufficient"; the open cells of the matrix define a research agenda.

## What's Novel

- First survey framing MCP/agent security through the *execution layer's* properties rather than conventional software assumptions — the insight that blockchain semantics (not the agent code) govern attack consequences.
- The risk-mapping matrix tying attack classes to amplifiers and residual gaps is a reusable framework for scoping agent-security work in financial/Web3 contexts.

## My Connection (to Manny's work)

Directly extends Manny's MCP threat modeling: any agent with a signing key or payment tool inherits the Web3 threat model — irreversibility means prompt-injection-to-action is a loss event, not a containment exercise. The matrix is a ready-made checklist for auditing MCP servers that touch financial state.

## What I Learned (plain English)

- The same prompt injection that's annoying in a chat is catastrophic on a blockchain: no undo, no chargeback, and agents act continuously and autonomously.
- Defense depth is currently awful: even combined, defenses stop under a third of attacks, and model refusal is almost useless (under 3%).
- If you deploy agents that can sign or spend, the security bar isn't "prevent the injection" — it's "make the irreversible action require human confirmation outside the agent's control."
