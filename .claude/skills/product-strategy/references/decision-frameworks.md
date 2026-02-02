# Decision Frameworks

## Purpose

Structured thinking for product decisions. Good frameworks:
- Make trade-offs explicit
- Create shared understanding
- Document rationale for future reference
- Reduce decision fatigue

---

## One-Way vs Two-Way Doors

Classify decisions by reversibility (from Jeff Bezos).

### Framework

```
┌─────────────────────────────────────────────────────────────────┐
│ ONE-WAY DOORS (Type 1)              TWO-WAY DOORS (Type 2)      │
│ Irreversible or very costly         Easily reversible           │
│ to reverse                                                      │
│                                                                 │
│ • Architecture choices              • Feature experiments       │
│ • Platform/language decisions       • UI changes                │
│ • Public API contracts              • Pricing tests             │
│ • Major pricing changes             • Copy/messaging            │
│ • Acquisitions/partnerships         • Internal tooling          │
│                                                                 │
│ APPROACH: Deliberate                APPROACH: Bias to action    │
│ • Gather more data                  • Decide quickly            │
│ • Involve more stakeholders         • Empower teams             │
│ • Document thoroughly               • Learn and iterate         │
│ • Plan carefully                    • Don't over-analyze        │
└─────────────────────────────────────────────────────────────────┘
```

### Application

Before any decision, ask:
1. What would it cost to reverse this decision?
2. How long would reversal take?
3. What damage occurs if we're wrong?

If costs are low → Treat as Type 2, move fast
If costs are high → Treat as Type 1, be deliberate

---

## Trade-off Matrix

Make trade-offs explicit when choosing between options.

### Template

```markdown
## Decision: [What we're deciding]

### Options
1. [Option A]: [Brief description]
2. [Option B]: [Brief description]
3. [Option C]: [Brief description]

### Criteria (weighted by importance)
| Criteria | Weight | Option A | Option B | Option C |
|----------|--------|----------|----------|----------|
| User value | 30% | 4 | 3 | 5 |
| Effort | 25% | 3 | 5 | 2 |
| Risk | 20% | 4 | 4 | 3 |
| Strategic fit | 15% | 5 | 3 | 4 |
| Maintainability | 10% | 3 | 4 | 3 |
| **Weighted Score** | | **3.75** | **3.75** | **3.55** |

### Recommendation
[Option X] because [rationale beyond just the score]

### Risks & Mitigations
- [Risk 1]: [Mitigation]
- [Risk 2]: [Mitigation]
```

---

## DACI Framework

Clarify roles for decision-making.

### Roles

| Role | Definition | Responsibility |
|------|------------|----------------|
| **D**river | Leads the decision process | Gathers input, drives to resolution |
| **A**pprover | Has final say | Makes the call, accountable for outcome |
| **C**ontributors | Provide input | Share expertise and perspective |
| **I**nformed | Need to know | Kept updated on decision |

### Template

```markdown
## Decision: [Title]

### DACI
| Role | Person |
|------|--------|
| Driver | [Name] |
| Approver | [Name] |
| Contributors | [Names] |
| Informed | [Names/Teams] |

### Timeline
- Input due: [Date]
- Decision by: [Date]
- Communication: [Date]

### Context
[Background and why this decision is needed]

### Options
[List options being considered]

### Recommendation
[Driver's recommendation with rationale]

### Decision
[Final decision once made]

### Rationale
[Why this option was chosen]
```

---

## Eisenhower Matrix

Prioritize by urgency and importance.

### Framework

```
                      URGENT
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    │   DO FIRST         │   SCHEDULE         │
    │                    │                    │
    │ • Crisis items     │ • Strategic work   │
    │ • Deadlines        │ • Planning         │
    │ • Emergencies      │ • Relationships    │
    │                    │                    │
IMP ├────────────────────┼────────────────────┤ NOT
ORT │                    │                    │ IMP
ANT │   DELEGATE         │   ELIMINATE        │ ORT
    │                    │                    │ ANT
    │ • Interruptions    │ • Time wasters     │
    │ • Some meetings    │ • Busy work        │
    │ • Some emails      │ • Some meetings    │
    │                    │                    │
    └────────────────────┼────────────────────┘
                         │
                    NOT URGENT
```

### Application to Product

- **Do First (Urgent + Important):** Production bugs, security issues
- **Schedule (Important, Not Urgent):** Technical debt, user research
- **Delegate (Urgent, Not Important):** Support escalations, minor requests
- **Eliminate (Neither):** Feature requests with no user evidence

---

## Hypothesis-Driven Decisions

Frame decisions as experiments.

### Template

```markdown
## Hypothesis: [Title]

### We believe that
[Proposed action or change]

### For
[Target user segment]

### Will result in
[Expected outcome]

### We will know this is true when
[Measurable indicator]

### We will test this by
[Experiment design]

### Timeline
- Experiment start: [Date]
- Minimum runtime: [Duration]
- Decision point: [Date]

### Success criteria
- Proceed if: [Threshold]
- Stop/pivot if: [Threshold]
- Gather more data if: [Conditions]
```

---

## Pre-mortem Analysis

Imagine failure before it happens.

### Process

1. **Assume failure:** "It's 6 months from now, and this project failed spectacularly."
2. **Generate reasons:** Each person writes why it failed
3. **Share and cluster:** Group similar failure modes
4. **Prioritize:** Rank by likelihood and severity
5. **Mitigate:** Create plans to prevent top failures

### Template

```markdown
## Pre-mortem: [Project Name]

### Scenario
It's [future date]. [Project] has failed. What went wrong?

### Potential Failure Modes

| Failure Mode | Likelihood | Severity | Mitigation |
|--------------|------------|----------|------------|
| [Failure 1] | High | High | [Action] |
| [Failure 2] | Medium | High | [Action] |
| [Failure 3] | High | Medium | [Action] |

### Early Warning Signs
- [Signal that indicates we're heading toward failure 1]
- [Signal that indicates we're heading toward failure 2]

### Checkpoints
- [Date]: Review [Metric] for signs of [Failure mode]
```

---

## Decision Log

Document decisions for future reference.

### Template

```markdown
## Decision Log: [Project/Product]

### [Date] - [Decision Title]

**Context:** [Why this decision was needed]

**Options Considered:**
1. [Option A]
2. [Option B]
3. [Option C]

**Decision:** [What was decided]

**Rationale:** [Why this option]

**Trade-offs Accepted:** [What we gave up]

**Revisit Trigger:** [Conditions that would reopen this decision]

**Participants:** [Who was involved]

---

### [Date] - [Next Decision]
...
```

---

## Decision Quality Checklist

Before finalizing any significant decision:

### Information Quality
- [ ] Do we have the information needed to decide?
- [ ] Have we sought out disconfirming evidence?
- [ ] Are we relying on assumptions? Are they validated?

### Process Quality
- [ ] Have the right people been involved?
- [ ] Has there been genuine debate, not just consensus?
- [ ] Is the driver clear about who approves?

### Outcome Framing
- [ ] Can we articulate what success looks like?
- [ ] Do we know how we'll measure it?
- [ ] Have we identified how we'd know we're wrong?

### Reversibility Check
- [ ] Is this a one-way or two-way door?
- [ ] What's the cost of reversal?
- [ ] Should we be moving faster or slower?

### Documentation
- [ ] Is the decision and rationale documented?
- [ ] Do stakeholders know what was decided?
- [ ] Is there a plan to revisit if conditions change?
