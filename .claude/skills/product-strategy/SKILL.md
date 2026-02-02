---
name: product-strategy
description: Strategic product thinking, roadmap planning, competitive analysis, and PRD generation. Use for defining product vision, prioritizing features, analyzing competitors, synthesizing user research, and creating formal product specifications.
---

<why_now>
## Why Product Strategy Matters

Building the right thing is harder than building the thing right. Product strategy ensures you're solving real problems for real users before writing code. The methodology:

- **Vision-driven development** — Every feature traces back to user needs and business goals
- **Evidence-based decisions** — Research and data inform priorities, not opinions
- **Clear scope boundaries** — Know what you're building AND what you're not building
- **Measurable outcomes** — Success metrics defined before implementation

As Marty Cagan says: "Fall in love with the problem, not the solution." Product strategy keeps teams focused on problems worth solving.
</why_now>

<core_principles>
## Core Principles

### 1. Problem-First Thinking

Start with problems, not solutions:
- What user pain are we addressing?
- How big is this problem? For whom?
- What happens if we don't solve it?

**Test:** Can you articulate the problem without mentioning your solution?

### 2. Jobs-to-be-Done

Users "hire" products to accomplish jobs:
- Functional jobs: Tasks they need to complete
- Emotional jobs: How they want to feel
- Social jobs: How they want to be perceived

**Test:** What job is the user hiring your product to do?

### 3. Scope Discipline

Ruthlessly prioritize:
- Must-have: Product doesn't work without it
- Should-have: Significant value, but not blocking
- Won't-have: Explicitly out of scope (for now)

**Test:** If you can't say "no" to something, you can't say "yes" to anything.

### 4. Outcome Over Output

Measure success by outcomes, not features shipped:
- What behavior change do we expect?
- How will we know it worked?
- What's our hypothesis?

**Test:** Can you define success without counting features?

### 5. Continuous Discovery

Strategy is ongoing, not one-time:
- Regular user research
- Metrics monitoring
- Competitive awareness
- Iteration based on learning

**Test:** When did you last talk to a user?
</core_principles>

<intake>
## What product strategy work do you need help with?

1. **Create/update PRD** — Generate a Product Requirements Document from vision and research
2. **Build roadmap** — Create prioritized roadmap using Now/Next/Later or RICE scoring
3. **Competitive analysis** — Analyze competitors and identify differentiation opportunities
4. **Synthesize research** — Turn user interviews, surveys, and feedback into actionable insights
5. **Define success metrics** — Establish KPIs, OKRs, or success criteria
6. **Scope decisions** — Help prioritize Must/Should/Won't have features
7. **Decision framework** — Work through a specific product decision with structured thinking

**Wait for response before proceeding.**
</intake>

<routing>
| Response | Action |
|----------|--------|
| 1, "prd", "requirements", "spec" | Read [prd-templates.md](./references/prd-templates.md), offer to run `/prd` command |
| 2, "roadmap", "prioritize", "plan" | Read [roadmap-frameworks.md](./references/roadmap-frameworks.md), offer to run `/roadmap` |
| 3, "competitive", "competitor", "market" | Read [competitive-analysis.md](./references/competitive-analysis.md), offer to run `/competitive` |
| 4, "research", "user", "interview", "feedback" | Read [user-research-synthesis.md](./references/user-research-synthesis.md) |
| 5, "metrics", "okr", "kpi", "success" | Read [prd-templates.md](./references/prd-templates.md) (metrics section) and [decision-frameworks.md](./references/decision-frameworks.md) |
| 6, "scope", "prioritize", "must have", "mvp" | Read [roadmap-frameworks.md](./references/roadmap-frameworks.md) (prioritization section) |
| 7, "decision", "trade-off", "choose" | Read [decision-frameworks.md](./references/decision-frameworks.md) |

**After reading references, apply frameworks to the user's specific context.**
</routing>

<parallel_agents>
## Parallelized Sub-Agents

For comprehensive strategy work, launch these agents **in parallel**:

### Discovery Phase (Parallel)
```
┌─────────────────────────┐  ┌─────────────────────────┐
│ competitive-analyst     │  │ repo-research-analyst   │
│ → Analyzes market       │  │ → Understands codebase  │
│   landscape             │  │   current state         │
└───────────┬─────────────┘  └───────────┬─────────────┘
            │                            │
            └──────────┬─────────────────┘
                       ▼
         ┌─────────────────────────┐
         │ prd-generator           │
         │ → Synthesizes into PRD  │
         └─────────────────────────┘
```

### Planning Phase (Sequential)
```
┌─────────────────────────┐
│ roadmap-synthesizer     │
│ → Creates prioritized   │
│   roadmap from PRD      │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────────────────────────┐
│ Development workflow                        │
│                                             │
│ /dev → Parallel startup with PRD context    │
│     → Work loop with blog updates           │
│     → Iteration until features complete     │
└─────────────────────────────────────────────┘
```
</parallel_agents>

<prd_workflow>
## PRD Generation Workflow

### Step 1: Gather Context (Parallel Agents)

Launch 3 agents simultaneously:

**Agent 1: Existing Plans**
```
subagent_type: Explore
prompt: "Find and summarize any existing product documents:
- docs/product/ or similar directories
- README product descriptions
- Any PRD, spec, or requirements files
Return: Product vision, stated goals, existing scope"
```

**Agent 2: Current Implementation**
```
subagent_type: Explore
prompt: "Analyze what's currently built:
- What features/pages exist?
- What's functional vs stubbed?
- What patterns are established?
Return: Gap analysis (planned vs built)"
```

**Agent 3: Technical Constraints**
```
subagent_type: Explore
prompt: "Identify technical constraints:
- Tech stack and dependencies
- Integration requirements
- Performance/scale considerations
Return: Technical boundaries for product decisions"
```

### Step 2: Clarify with User

Use AskUserQuestion to resolve:
- Target users and their primary jobs-to-be-done
- Success definition (what does "working" look like?)
- Scope boundaries (what's explicitly NOT in scope?)
- Timeline/phase expectations

### Step 3: Generate PRD

Write to `docs/product/PRD.md` with structure from [prd-templates.md](./references/prd-templates.md).
</prd_workflow>

<reference_index>
## Reference Files

All references in `references/`:

**Strategy Frameworks:**
- [roadmap-frameworks.md](./references/roadmap-frameworks.md) — Now/Next/Later, RICE scoring, MoSCoW prioritization
- [competitive-analysis.md](./references/competitive-analysis.md) — Porter's Five Forces, SWOT, feature matrices
- [decision-frameworks.md](./references/decision-frameworks.md) — Trade-off matrices, reversibility analysis, one-way doors

**Research & Discovery:**
- [user-research-synthesis.md](./references/user-research-synthesis.md) — Jobs-to-be-done, personas, journey mapping, affinity diagramming

**Documentation:**
- [prd-templates.md](./references/prd-templates.md) — PRD structure, feature specs, success metrics
</reference_index>

<commands>
## Available Commands

- `/prd` — Generate or update Product Requirements Document
- `/roadmap` — Create prioritized product roadmap
- `/competitive` — Run competitive analysis
</commands>

<anti_patterns>
## Anti-Patterns

### Strategy Anti-Patterns

**Solution-first thinking** — Starting with "let's build X" instead of "users need Y"
```
✗ "We should add AI features because competitors have them"
✓ "Users struggle with X task; AI could reduce effort by Y%"
```

**Feature soup** — No clear prioritization, everything is "important"
```
✗ PRD with 50 "must-have" features
✓ 3-5 must-haves, clear should-have and won't-have lists
```

**Vanity metrics** — Measuring output instead of outcomes
```
✗ "Success = ship 10 features this quarter"
✓ "Success = 20% reduction in user task completion time"
```

**Scope creep acceptance** — Saying yes to everything
```
✗ "Sure, we can add that too"
✓ "That's a great idea for v2; for v1, we're focused on X"
```

### PRD Anti-Patterns

**Novel-length specs** — PRDs so long no one reads them
```
✗ 50-page PRD with exhaustive detail
✓ 2-3 page PRD with links to detailed specs as needed
```

**Missing "won't have"** — No explicit scope boundaries
```
✗ PRD only says what you will build
✓ PRD explicitly lists what you won't build (this release)
```

**No success criteria** — No way to know if it worked
```
✗ "Users can upload files"
✓ "Users can upload files <10MB in <3 seconds; 80% complete uploads without retry"
```
</anti_patterns>

<success_criteria>
## Success Criteria

You've done good product strategy when:

### Clarity
- [ ] Problem is clearly articulated without referencing solution
- [ ] Target users are specific (not "everyone")
- [ ] Success metrics are measurable and time-bound
- [ ] Scope boundaries are explicit (must/should/won't have)

### Alignment
- [ ] Team agrees on priorities
- [ ] Stakeholders understand trade-offs
- [ ] Technical constraints inform scope decisions
- [ ] Roadmap reflects strategic priorities

### Documentation
- [ ] PRD exists and is current
- [ ] Roadmap is accessible and updated
- [ ] Decisions are documented with rationale
- [ ] Research findings are synthesized and shared

### The Ultimate Test

**Can a new team member understand what you're building and why in 10 minutes?**
**Can they explain what you're NOT building and why?**

If yes, you've done good product strategy.
</success_criteria>
