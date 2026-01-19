# Epistemology Layer

> **How Workshop learns from experience**

---

## Overview

The epistemology layer is Workshop's memory system—not a flat database, but a structured knowledge lifecycle with validation gates, confidence scoring, and provenance tracking.

This document describes:
1. The **knowledge lifecycle** (types and transitions)
2. The **auto-learning pipeline** (how evidence becomes skills)
3. The **components** that implement it
4. The **CLI commands** for interacting with knowledge

---

## The Knowledge Lifecycle

Knowledge in Workshop has a type that reflects its epistemic status:

```
hypothesis → observation → pattern → justified_belief → validated_knowledge → wisdom
                                            ↓
                                    [Promotion Gate]
                                            ↓
                                         Skill
```

### Type Definitions

| Type | Confidence | Meaning |
|------|------------|---------|
| `hypothesis` | 0.1 - 0.3 | Untested idea, worth exploring |
| `observation` | 0.3 - 0.5 | Seen once, noted but not validated |
| `pattern` | 0.5 - 0.7 | Recurring across multiple observations |
| `justified_belief` | 0.7 - 0.85 | Evidence supports it, validated in multiple contexts |
| `validated_knowledge` | 0.85 - 0.95 | Proven reliable, documented failure modes |
| `wisdom` | 0.95+ | Deep principle, rarely changes |

### Transitions

Knowledge doesn't jump levels. Each transition requires evidence:

- **hypothesis → observation**: Actually encountered in practice
- **observation → pattern**: Seen multiple times, clustered by similarity
- **pattern → justified_belief**: Validated in 3+ independent contexts
- **justified_belief → validated_knowledge**: Failure modes documented, consistently reliable
- **validated_knowledge → wisdom**: Fundamental principle that explains other knowledge

---

## The Auto-Learning Pipeline

Workshop automates the journey from raw experience to reusable skills:

```
┌─────────────────────────────────────────────────────────────────┐
│                      EVIDENCE SOURCES                           │
├─────────────────────┬─────────────────────┬─────────────────────┤
│  CC Session Reports │   WS Action Logs    │    Manual Entry     │
│  (Workflows A & B)  │   (Workflow C)      │    (Fallback)       │
└──────────┬──────────┴──────────┬──────────┴──────────┬──────────┘
           │                     │                     │
           ▼                     ▼                     ▼
┌─────────────────────────────────────────────────────────────────┐
│                    EVIDENCE COLLECTION                          │
│                                                                 │
│  • EvidenceIngestor — Parses CC session reports                 │
│  • SessionObserver — Captures WS execution events               │
│  • ObservationStore — Persists with deduplication               │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PATTERN DETECTION                            │
│                                                                 │
│  • Clusters related observations                                │
│  • Calculates confidence from occurrences + signal + diversity  │
│  • Proposes patterns for review                                 │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PATTERN VALIDATION                           │
│                                                                 │
│  • Checks promotion criteria (contexts, confidence, failures)   │
│  • Promotes eligible patterns to justified_belief               │
│  • Flags patterns needing more evidence                         │
└─────────────────────────────────┬───────────────────────────────┘
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────────┐
│                    PROMOTION GATE                               │
│                                                                 │
│  • Final validation for skill graduation                        │
│  • Requires documented failure modes                            │
│  • Creates versioned, auditable skills                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## Pipeline Stages

### Stage 1: Session → Evidence

**Components:**
- `SessionObserver` — EventEmitter-based capture during WS execution
- `ObservationStore` — YAML-based storage with deduplication
- `EvidenceIngestor` — Parses CC session reports into observations

**What gets captured:**
- Success/failure outcomes
- Tool usage patterns
- Model performance data
- Workarounds discovered
- Decisions made

**Observation Structure:**

```yaml
id: obs_2026_01_19_001
category: success           # success | failure | pattern | tool_usage | model_perf | workaround | decision
claim: "Cloudflare deployment succeeded with absolute paths"
signal_strength: moderate   # weak | moderate | strong
occurrences: 3
first_seen: "2026-01-19T10:30:00Z"
last_seen: "2026-01-19T22:15:00Z"
sessions:
  - "session_2026_01_19_morning"
  - "session_2026_01_19_evening"
tags:
  - deployment
  - cloudflare
  - paths
model_provenance:
  provider: anthropic
  model_id: claude-sonnet-4-20250514
promoted: false
```

**CLI Commands:**

```bash
# List observations
workshop evidence list
workshop evidence list --category failure
workshop evidence list --unpromoted

# View details
workshop evidence show obs_2026_01_19_001

# Statistics
workshop evidence stats

# Promotion candidates (3+ occurrences OR strong signal)
workshop evidence candidates

# Promote to knowledge entry
workshop evidence promote obs_2026_01_19_001

# Clean up old weak evidence
workshop evidence prune --older-than 30d --signal weak
```

---

### Stage 2: Evidence → Pattern

**Component:** `PatternDetector`

**How clustering works:**

1. Sort observations by occurrence count (highest first)
2. For each unassigned observation:
   - Find similar observations (same category + text similarity ≥30%)
   - Lower threshold if same model or same tool
   - Group into cluster if meets minimum observations
3. Calculate cluster metrics:
   - Total occurrences across observations
   - Average signal strength
   - Dominant model/tool (if >50% share)
   - Common tags (if >50% share)
4. Generate proposal if meets thresholds

**Confidence Calculation:**

```
confidence = min(0.95,
  min(0.3, observations.length * 0.1) +    // Base from observation count
  min(0.3, totalOccurrences * 0.02) +      // Boost from occurrences
  avgSignal * 0.3 +                         // Boost from signal strength
  (uniqueSessions >= 2 ? 0.1 : 0)           // Boost from session diversity
)
```

**Recommendation Strength:**

| Strength | Criteria |
|----------|----------|
| `strong` | confidence ≥ 0.7 AND occurrences ≥ 5 |
| `moderate` | confidence ≥ 0.5 AND occurrences ≥ 3 |
| `weak` | Everything else meeting thresholds |

**CLI Commands:**

```bash
# Detect patterns from observations
workshop patterns-detect

# With options
workshop patterns-detect --min-obs 2 --min-occ 3 --min-conf 0.4
workshop patterns-detect --category failure
workshop patterns-detect --include-weak
workshop patterns-detect --verbose

# Accept proposals
workshop patterns-detect --accept 0        # Accept by index
workshop patterns-detect --accept-strong   # Auto-accept all strong
```

---

### Stage 3: Pattern → Justified Belief

**Component:** `PatternValidator`

**Promotion Criteria:**

| Criterion | Threshold | Rationale |
|-----------|-----------|-----------|
| Context Independence | ≥3 contexts | Pattern works across situations |
| Failure Documentation | ≥1 failure mode | Known limitations documented |
| Confidence Threshold | ≥75% | High certainty before promotion |
| Temporal Validity | ≤30 days since validation | Recently validated, still relevant |

**Verdict Types:**

- `promote` — All criteria pass, ready for promotion
- `not_ready` — Critical criteria fail (context/confidence)
- `promote_with_warnings` — Non-critical criteria fail (temporal/failure modes)

**CLI Commands:**

```bash
# Validate all patterns
workshop episteme validate

# Verbose output
workshop episteme validate --verbose

# Auto-promote eligible
workshop episteme validate --auto-promote
```

---

### Stages 4-5: Justified Belief → Skill

**Component:** `PromotionGate` (existing)

The promotion gate applies the same criteria with additional requirements for skill graduation:

- Complete provenance chain
- Documented failure modes with mitigations
- Version tracking
- Audit trail

**CLI Commands:**

```bash
# Check promotion eligibility
workshop episteme promote ke_2026_01_19_001 --dry-run

# Promote to skill
workshop episteme promote ke_2026_01_19_001
```

---

## Evidence Sources

### Source A: CC Session Reports

When Claude Code supervises Workshop sessions, the session report captures:
- Accomplishments (→ success observations)
- Failures (→ failure observations)
- Technical details (→ tool_usage, model_perf)
- Learnings (→ pattern observations)
- Workarounds (→ workaround observations)

**Ingestion:**

```bash
# Single report
workshop evidence ingest docs/session-reports/SESSION-REPORT-2026-01-19.md

# All reports in directory
workshop evidence ingest docs/session-reports/
```

### Source B: WS Action Logs

During `workshop action` execution, `SessionObserver` automatically captures:
- Tool executions
- Model responses
- Errors and recoveries
- Success/failure outcomes

This happens automatically—no manual step required.

### Source C: Manual Entry

For knowledge that doesn't come from automated sources:

```bash
workshop evidence add \
  --category pattern \
  --claim "API rate limits require exponential backoff" \
  --signal strong \
  --tags "api,reliability,patterns"
```

---

## Knowledge Entry Structure

Once observations graduate to knowledge entries:

```yaml
id: ke_2026_01_19_001
type: pattern                    # hypothesis | observation | pattern | justified_belief | validated_knowledge | wisdom
status: validated                # candidate | emerging | validated | superseded

claim: "Gemini models fail at file write operations requiring path resolution"
detail: |
  When using Gemini 2.5 Flash for file operations, the model frequently
  fails to resolve relative paths correctly, leading to write failures.
  
implications:
  - Use absolute paths when routing file operations to Gemini
  - Consider escalating file operations to Claude models
  - Monitor for improvements in future Gemini versions

domain: patterns/failure/gemini
tags:
  - gemini
  - file-operations
  - routing

confidence: 0.72
granularity: task                # action | task | workflow | project

evidence:
  - type: direct_experience
    description: "Observed during Sip & Sing deployment"
    date: "2026-01-15"
    weight: 0.8
  - type: observation
    description: "Repeated in Workshop platform development"
    date: "2026-01-19"
    weight: 0.9

validation:
  independent_contexts: 3
  last_validated: "2026-01-19"
  validation_contexts:
    - "Sip & Sing PWA"
    - "Workshop CLI"
    - "VisionSF website"

failure_modes:
  - condition: "File already exists with correct content"
    consequence: "False positive failure detection"
    mitigation: "Check file content before marking as failure"

provenance:
  created_by: "PatternDetector"
  created_at: "2026-01-19T22:30:00Z"
  context: "Auto-learning pipeline"
  discovery_method: automated
  derived_from:
    - obs_2026_01_19_001
    - obs_2026_01_19_002
    - obs_2026_01_19_003

skill_promotion_status:
  status: not_candidate          # not_candidate | nominated | pending_review | promoted | rejected
```

---

## Storage

### Personal Knowledge Store

Location: `~/.workshop/epistemology/`

```
~/.workshop/epistemology/
├── knowledge.yaml      # Knowledge entries
└── observations.yaml   # Raw observations
```

### Project Knowledge Store

Location: `.workshop/epistemology/`

```
.workshop/epistemology/
├── knowledge.yaml      # Project-specific knowledge
└── observations.yaml   # Project-specific observations
```

Project knowledge takes precedence over personal knowledge for project-specific patterns.

---

## CLI Reference

### Evidence Commands

```bash
workshop evidence list [--category <cat>] [--unpromoted] [--promoted]
workshop evidence show <id>
workshop evidence stats
workshop evidence candidates
workshop evidence promote <id>
workshop evidence prune [--older-than <days>] [--signal <strength>]
workshop evidence add --category <cat> --claim <text> [--signal <strength>] [--tags <tags>]
workshop evidence ingest <file|directory>
```

### Pattern Commands

```bash
workshop patterns-detect [options]
  --min-obs <n>       Minimum observations to form cluster (default: 2)
  --min-occ <n>       Minimum total occurrences (default: 3)
  --min-conf <n>      Minimum confidence 0-1 (default: 0.4)
  -c, --category      Filter by category
  --include-weak      Include weak signal observations
  --include-promoted  Include already promoted observations
  --accept <index>    Accept proposal by index
  --accept-strong     Auto-accept all strong recommendations
  -v, --verbose       Verbose output
```

### Knowledge Commands

```bash
workshop episteme list [--type <type>] [--domain <domain>]
workshop episteme show <id>
workshop episteme stats
workshop episteme candidates
workshop episteme validate [--verbose] [--auto-promote]
workshop episteme promote <id> [--dry-run]
workshop episteme-add
workshop episteme-update <id>
workshop episteme-delete <id>
workshop episteme-stores
workshop episteme-init
workshop episteme-export [file]
workshop episteme-import <file>
```

---

## Design Principles

### 1. Provenance Is Non-Negotiable

Every piece of knowledge must answer: Who discovered this? When? In what context? How was it validated?

### 2. Confidence Decays

Knowledge that hasn't been validated recently loses confidence. Stale knowledge is suspect knowledge.

### 3. Failure Modes Are Required

Knowledge without documented failure modes is incomplete. Knowing when something *doesn't* work is as important as knowing when it does.

### 4. Automate Collection, Gate Promotion

Evidence collection should be automatic. Promotion to skills should be gated by validation criteria.

### 5. The Child Learns From The Parent

CC (supervisor) generates session reports. WS (worker) ingests them. The parent's experience becomes the child's knowledge.

---

## The Learning Loop

```
CC supervises WS session
        ↓
Session report generated
        ↓
workshop evidence ingest <report>
        ↓
Observations stored with deduplication
        ↓
workshop patterns-detect --accept-strong
        ↓
Patterns created from clustered observations
        ↓
workshop episteme validate
        ↓
Patterns promoted to justified_belief
        ↓
workshop episteme promote <id>
        ↓
Knowledge graduates to skill
        ↓
WS uses skill in future sessions
        ↓
Skill validated through use
        ↓
(Loop continues)
```

---

## Further Reading

- [Architecture Overview](./ARCHITECTURE.md) — The four-layer system
- [Supervision Protocol](./SUPERVISION.md) — L1-L4.5 intervention levels *(coming soon)*
- [Promotion Gate](./PROMOTION-GATE.md) — Validation criteria details *(coming soon)*

---

<p align="center">
  <i>"The model is frozen. The knowledge is not."</i>
</p>
