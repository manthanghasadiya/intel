# Self-State Attacks on Self-Hosted AI Agents: How Far Can OS Defenses Go?

**Authors:** Yimeng Chen, Nathanaël Denis, Roberto Di Pietro, Jürgen Schmidhuber
**Institution:** Not explicitly stated (includes Schmidhuber — AI Initiative, King Abdullah University of Science and Technology or similar)
**Published:** July 20, 2026
**Link:** https://arxiv.org/abs/2607.17986
**Code:** Not specified

---

## Problem Statement

Self-hosted AI agents read and write their own memory and configuration files to function. An attacker can compromise an agent by corrupting its **own state files** — memory snapshots, config files, persisted state — using only **legitimate OS system calls** (no exploit needed). The authors call this "self-state attacks." The core question: **how resilient are OS-level defenses against this class of attack?**

This is fundamentally different from traditional binary exploitation or memory corruption — the attacker doesn't need to exploit a vulnerability. They simply write to the agent's own state files via normal filesystem operations.

## Methodology

1. **Formal attack space definition**: Four-axis characterization:
   - **Target**: which self-state file is attacked (instruction file, config file, memory file)
   - **Mechanism**: how the state is modified (overwrite, append, delete, corrupt)
   - **Granularity**: scope of modification (single field, full file, cross-file)
   - **Temporal**: when the modification takes effect (next load, immediate, delayed)

2. **Trace collection**: Collected live activity traces from a representative self-hosted AI agent running across distinct workload profiles.

3. **Attack instantiation**: Realized the attack space as a **23-cell matrix** with **43 concrete operations** on real self-state files, injected into the collected traces.

4. **Defense evaluation**: Tested both canonical OS defenses and workload-conditioned defenses:
   - **Prevention**: Mandatory Access Control (MAC), file permissions, SELinux/AppArmor profiles
   - **Detection**: File integrity monitoring, auditd, eBPF-based anomaly detection
   - **Recovery**: Periodic snapshots and backup restoration

## Key Results

| Defense Layer | Attack Cells Blocked | Residual |
|---------------|---------------------|----------|
| Access-control prevention (instruction + config layers) | Most instruction & config attacks | Memory-layer attacks bypass |
| Workload-conditioned detection (memory layer) | Most memory-layer attacks | Small residual set |
| Periodic backup (recovery) | Enables recovery from all detected attacks | Structurally undetectable subset remains |

**Key finding:** A **layered defense stack** — access-control prevention on instruction/config layers, workload-conditioned detection on memory layer, periodic backup for recovery — is effective on **most attack cells**. However, a **small residual attack surface remains structurally indistinguishable at the OS level** — these attacks look identical to normal agent operation.

## What's Novel

1. **Novel attack class**: "Self-state attacks" — compromising agents by modifying their own state via legitimate OS operations — is a formally defined new threat category
2. **Four-axis formalization**: Provides a structured way to think about agent state attacks beyond ad-hoc enumeration
3. **Workload-conditioned detection**: Novel idea that detection effectiveness depends on the agent's workload profile — what's "normal" varies by task
4. **Structural indistinguishability proof**: Demonstrates that some self-state attacks are **fundamentally undetectable** at the OS level because they're identical to legitimate agent writes
5. **Practical evaluation**: 23-cell attack matrix with 43 concrete operations on real agent traces

## My Connection

This is **extremely relevant** to Manny's mcpsec work — MCP servers are self-hosted AI agents:

- **MCP servers maintain state**: Configuration files, tool definitions, memory/persistence layers, authentication tokens. All of these are self-state files.
- **Self-state attacks bypass traditional defenses**: Since writes happen through legitimate OS calls (the MCP server writing its own config), standard file integrity monitoring may not distinguish malicious from benign writes.
- **The 23-cell attack matrix** provides a ready-made test framework: Manny can enumerate which cells apply to mcpsec and test against them.
- **Structural indistinguishability** has implications for mcpsec's detection layer: some agent state tampering cannot be detected at the OS level, requiring application-level (MCP-layer) detection instead.
- **Agent memory corruption**: If an MCP server persists conversation history or tool-use results to disk, poisoning that memory is a self-state attack.

## What I Learned

- Self-state attacks are a **fundamentally different threat model** from traditional exploits — they use legitimate OS operations against legitimate agent files
- A **layered defense works for most cases** but **100% coverage is impossible** at the OS level — some attacks are structurally identical to normal agent operation
- The **workload profile matters** for detection accuracy: what's "normal" for an idle agent vs a busy one looks different
- **Application-layer defenses are necessary** for the residual undetectable attacks — this is where mcpsec's tool-level access control fills the gap
- The four-axis framework (Target × Mechanism × Granularity × Temporal) is a useful mental model for any agent security work
