# Workshop Methodology

> **A self-improving approach to AI-native software development**

---

## What This Is

Workshop is a methodology for building software with AI systems that **learn from experience**.

Unlike traditional AI coding assistants that start fresh every session, Workshop implements:

- **Persistent knowledge** that accumulates across projects
- **Formal validation** before knowledge becomes reusable skills
- **Graduated supervision** that decreases as the system matures
- **Cross-model memory** that works regardless of which AI you use

The result: an AI development workflow that gets measurably better with every project completed.

---

## What This Is NOT

- ❌ A product you can install today
- ❌ Production-ready code
- ❌ A framework or library
- ❌ Another AI coding assistant

This repository documents the **methodology and architecture**—the thinking behind Workshop. Implementation details are not included.

---

## The Core Insight

> **Accumulated context beats model capabilities.**

AI models will keep improving. But the patterns specific to *your* codebase, *your* team's conventions, *your* domain's edge cases—no model update captures those.

Workshop creates infrastructure for **institutional AI memory**:
- Knowledge entries with confidence scores
- Validation gates preventing premature generalization  
- Skills that link back to evidence justifying their existence
- Provenance tracking so you know where recommendations came from

The model powering Workshop matters less than the knowledge it has accumulated.

---

## Architecture Overview

Workshop is built on a four-layer architecture:

```
┌─────────────────────────────────────────┐
│           AGENT LAYER                   │  Decisions & Routing
│         (Practical Reason)              │
├─────────────────────────────────────────┤
│           SKILLS LAYER                  │  Validated Procedures
│        (Procedural Knowledge)           │
├─────────────────────────────────────────┤
│        EPISTEMOLOGY LAYER               │  Knowledge with Provenance
│         (Justified Belief)              │
├─────────────────────────────────────────┤
│            MCP LAYER                    │  External Connections
│       (Perception & Action)             │
└─────────────────────────────────────────┘
```

**[→ Detailed Architecture](./ARCHITECTURE.md)**

---

## The Supervision Model

Workshop uses a **parent-child supervision model**:

- **Senior AI (CC)** supervises complex operations
- **Junior AI (WS)** executes tasks, learning from experience
- **Graduated protocol** (L1→L4) determines intervention level
- **Goal:** WS handles more autonomously over time

This mirrors how human teams develop junior members—through guided practice, not just instruction.

---

## The Promotion Gate

Not everything becomes a skill. Knowledge must pass validation:

| Criterion | Requirement |
|-----------|-------------|
| **Context Independence** | Validated in 3+ independent contexts |
| **Failure Documentation** | At least one failure mode documented |
| **Confidence Threshold** | 75% or higher |
| **Provenance** | Complete tracking of source and validation |

This prevents premature generalization. A pattern that worked once might be coincidence.

---

## Why We're Sharing This

We believe this approach will transform how AI-assisted development is done.

By sharing the methodology openly, we hope to:
- Advance the conversation beyond "AI as tool" to "AI as learning system"
- Invite critique and improvement from practitioners
- Establish shared vocabulary for this emerging space

**What we're sharing:** The concepts, architecture, and protocols.

**What remains operational:** Accumulated knowledge, trained skills, and implementation details. These represent years of learning that cannot be open-sourced—they must be built through practice.

---

## Current Status

🔬 **Early methodology documentation**

We are actively using this methodology in production at VisionSF. This documentation will expand based on community interest and our ongoing learnings.

---

## Who's Behind This

**[VisionSF](https://visionsf.com)** — An AI-native software development studio based in Silicon Valley.

We don't just use AI to assist development. AI *is* our development—supervised, validated, and continuously improving.

---

## Further Reading

- [Architecture Overview](./ARCHITECTURE.md) — The four-layer system in detail
- [Supervision Protocol](./SUPERVISION.md) — L1-L4 intervention levels *(coming soon)*
- [Epistemology Layer](./EPISTEMOLOGY.md) — Knowledge lifecycle *(coming soon)*
- [Promotion Gate](./PROMOTION-GATE.md) — Validation criteria *(coming soon)*

---

## Contributing

We welcome **ideas and critiques**, not code contributions (there's no code here).

- Open an issue to discuss methodology questions
- Share your own experiments with similar approaches
- Propose refinements to the architecture

---

## License

MIT — Use these ideas freely. Attribution appreciated.

---

<p align="center">
  <i>"Solve first, encode second."</i>
</p>
