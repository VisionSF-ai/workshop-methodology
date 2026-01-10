# Workshop Architecture

> **The four-layer system for self-improving AI development**

---

## Overview

Workshop separates concerns into four distinct layers, each with a specific responsibility:

```
┌─────────────────────────────────────────────────────────────────┐
│                        AGENT LAYER                              │
│                     (Practical Reason)                          │
│                                                                 │
│   • Decides what to do given a task                             │
│   • Routes to appropriate skills or models                      │
│   • Manages supervision protocol                                │
│   • Handles escalation and intervention                         │
├─────────────────────────────────────────────────────────────────┤
│                       SKILLS LAYER                              │
│                   (Procedural Knowledge)                        │
│                                                                 │
│   • Validated, reusable procedures                              │
│   • Triggered by context matching                               │
│   • Linked to epistemological evidence                          │
│   • Versioned and auditable                                     │
├─────────────────────────────────────────────────────────────────┤
│                    EPISTEMOLOGY LAYER                           │
│                     (Justified Belief)                          │
│                                                                 │
│   • Knowledge entries with confidence scores                    │
│   • Provenance tracking (who, when, how, why)                   │
│   • Validation history and failure modes                        │
│   • Promotion gate for skill graduation                         │
├─────────────────────────────────────────────────────────────────┤
│                        MCP LAYER                                │
│                   (Perception & Action)                         │
│                                                                 │
│   • File system operations                                      │
│   • API integrations                                            │
│   • Database connections                                        │
│   • External tool access                                        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Layer 1: MCP Layer (Perception & Action)

**Purpose:** Connect Workshop to the external world.

The MCP (Model Context Protocol) layer provides the hands and eyes of the system. It handles:

- **File operations** — Read, write, edit files in the project
- **Command execution** — Run builds, tests, deployments
- **API access** — Integrate with external services
- **Data retrieval** — Query databases, fetch resources

### Key Principle

The MCP layer is **capability**, not intelligence. It provides tools; other layers decide when and how to use them.

### Example MCP Tools

```
file_read      — Read file contents
file_write     — Create or overwrite files
file_edit      — Modify existing files (find/replace)
bash_execute   — Run shell commands
api_call       — Make HTTP requests
db_query       — Execute database queries
```

---

## Layer 2: Epistemology Layer (Justified Belief)

**Purpose:** Store knowledge with provenance and validation.

This is where Workshop diverges from typical AI systems. Knowledge isn't a flat database or vector store—it's a structured collection with:

### Knowledge Entry Structure

```yaml
id: ke_2026_01_05_001
type: validated_knowledge    # hypothesis | observation | validated_knowledge
claim: "Use absolute paths when WS struggles with directory context"
confidence: 0.85
domain: "workarounds/cwd"

provenance:
  discovered_by: "CC supervision session"
  discovered_date: "2026-01-04"
  context: "Sip & Sing project deployment"
  method: "Pattern emerged from repeated failures"

validation:
  contexts_validated: 3
  last_validated: "2026-01-05"
  failure_modes:
    - "Does not apply to relative imports within same directory"

relationships:
  derived_from: []
  led_to: ["ke_2026_01_05_003"]
  supersedes: []
```

### Knowledge Types (Hierarchy)

```
hypothesis           → Untested idea (low confidence)
    ↓
observation          → Seen once, noted
    ↓
justified_belief     → Evidence supports it
    ↓
validated_knowledge  → Proven across contexts
    ↓
wisdom               → Deep principle, rarely changes
```

### Why This Matters

Traditional AI: "I generated this answer based on training data."

Workshop: "I know this because of ke_2026_01_05_001, discovered during Sip & Sing, validated across 3 contexts, with documented failure modes."

**Provenance creates trust.**

---

## Layer 3: Skills Layer (Procedural Knowledge)

**Purpose:** Store validated, reusable procedures.

Skills are not prompts. They are complete procedures with:

- **Trigger conditions** — When should this skill activate?
- **Steps** — What actions to take
- **Validation links** — Which knowledge entries justify this skill?
- **Failure handling** — What to do when it doesn't work

### Skill Types

| Type | Can Modify? | Purpose |
|------|-------------|---------|
| **Sentinel** | Read-only | Observe, analyze, report |
| **Actuator** | Read-write | Take action, modify state |

### Skill Structure

```yaml
name: cloudflare-pages-deploy
type: actuator
version: 1.0.0

description: |
  Deploy static sites to Cloudflare Pages via API.
  Handles project creation, GitHub connection, and custom domains.

triggers:
  - "deploy to cloudflare"
  - "publish to pages"
  - "cloudflare deployment"

preconditions:
  - "~/.cloudflare_config exists with valid credentials"
  - "dist/ directory contains built assets"

steps:
  1. Read credentials from config
  2. Create Pages project via API
  3. Connect GitHub repository
  4. Configure custom domain
  5. Verify deployment status

evidence:
  - ke_2026_01_05_004  # Termux deployment workaround
  - ke_2026_01_05_005  # Cloudflare API pattern

failure_modes:
  - condition: "API returns 403"
    resolution: "Check token permissions for Pages and DNS"
  - condition: "Build fails with npm version error"
    resolution: "Delete package-lock.json, let CF regenerate"
```

### Promotion Gate

Skills are not created speculatively. They graduate from the epistemology layer:

```
Knowledge Entry
      ↓
Validated in 3+ contexts
      ↓
75%+ confidence
      ↓
Failure modes documented
      ↓
PROMOTION GATE
      ↓
Skill Created
```

This prevents premature generalization. A pattern that worked once is not a skill.

---

## Layer 4: Agent Layer (Practical Reason)

**Purpose:** Make decisions and manage execution.

The agent layer is where intelligence lives. It:

- **Analyzes tasks** — What is being asked?
- **Routes decisions** — Which skill applies? Which model?
- **Manages supervision** — When to escalate? When to intervene?
- **Coordinates execution** — Orchestrate multi-step workflows

### The Supervision Model

Workshop uses a parent-child supervision model:

```
┌─────────────────────────────────────────┐
│              CC (Parent)                │
│         Senior AI / Supervisor          │
│                                         │
│   • Handles complex decisions           │
│   • Intervenes when WS struggles        │
│   • Documents patterns to epistemology  │
│   • Teaches through guided practice     │
└────────────────┬────────────────────────┘
                 │
                 │ Supervises
                 ↓
┌─────────────────────────────────────────┐
│              WS (Child)                 │
│         Junior AI / Worker              │
│                                         │
│   • Executes tasks                      │
│   • Learns from experience              │
│   • Gains capabilities over time        │
│   • Eventually works autonomously       │
└─────────────────────────────────────────┘
```

### Supervision Protocol (L1-L4)

When WS fails at a task:

```
LEVEL 1: Refine Task
├── Break into smaller steps
├── Simplify language
├── Add constraints
└── Retry same model

         ↓ Still fails?

LEVEL 2: Switch Model
├── Try different AI model
├── Some tasks suit different models
└── Document which model worked

         ↓ Still fails?

LEVEL 3: Find Workaround
├── Discover alternative approach WS CAN execute
├── Document successful workaround pattern
└── Add to epistemology

         ↓ Workaround not possible?

LEVEL 4: CC Intervention
├── CC completes the task directly
├── Document what failed and why
├── Note capability gap for future improvement
└── This is a FAILURE of supervision, not success
```

### L4.5: Capability Gap Protocol

When failure is due to missing platform capability (not task complexity):

```
LEVEL 4.5: Capability Gap
├── Pause supervision
├── Document gap to epistemology
├── Fix platform capability
├── Test new capability
├── Resume supervision
└── WS retries with new capability
```

This distinguishes between "task was too hard" (L4) and "tool was missing" (L4.5).

---

## How Layers Interact

### Example: Deploying a Website

```
1. AGENT receives task: "Deploy to Cloudflare"
           ↓
2. AGENT checks SKILLS: cloudflare-pages-deploy exists
           ↓
3. AGENT checks EPISTEMOLOGY: Related knowledge loaded
           ↓
4. AGENT routes to WS with skill context
           ↓
5. WS executes skill steps using MCP tools
           ↓
6. Success → AGENT documents outcome
   Failure → AGENT escalates per supervision protocol
```

### Example: Learning New Pattern

```
1. WS fails at task (cwd issue)
           ↓
2. CC identifies pattern: "Absolute paths fix this"
           ↓
3. CC documents to EPISTEMOLOGY:
   - type: observation
   - confidence: 0.70
           ↓
4. Pattern succeeds in 2 more contexts
           ↓
5. PROMOTION GATE:
   - 3 contexts ✓
   - 75% confidence ✓
   - Failure modes documented ✓
           ↓
6. Knowledge graduates to SKILL
           ↓
7. Future tasks automatically use this skill
```

---

## Design Principles

### 1. Dynamic Beats Static

The model is frozen. The knowledge layer is not. Workshop's value accumulates through use.

### 2. Solve First, Encode Second

Skills are not written speculatively. They emerge from real problem-solving, validated across contexts.

### 3. Provenance Creates Trust

Every recommendation traces back to evidence. "I know this because..." is always answerable.

### 4. Graduated Autonomy

The system earns independence through demonstrated competence, not assumed capability.

### 5. Failure Is Data

Documented failures are more valuable than undocumented successes. Failure modes prevent future mistakes.

---

## Further Reading

- [README](./README.md) — Overview and context
- [Supervision Protocol](./SUPERVISION.md) — Detailed L1-L4.5 protocol *(coming soon)*
- [Epistemology Layer](./EPISTEMOLOGY.md) — Knowledge lifecycle *(coming soon)*
- [Promotion Gate](./PROMOTION-GATE.md) — Validation criteria *(coming soon)*

---

<p align="center">
  <i>"The code is the skeleton. The knowledge is the muscle."</i>
</p>

