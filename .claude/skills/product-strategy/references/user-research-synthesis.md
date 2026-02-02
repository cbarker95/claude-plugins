# User Research Synthesis

## Purpose

Transform raw research data into actionable product insights. Good synthesis:
- Identifies patterns across multiple data sources
- Separates signal from noise
- Prioritizes problems by impact
- Connects user needs to product opportunities

---

## Jobs-to-be-Done Framework

Users "hire" products to accomplish jobs. Focus on the job, not the product.

### Job Statement Format

```
When [situation], I want to [motivation], so I can [expected outcome].
```

### Examples

```markdown
## Job: Managing design feedback

When I receive design feedback from stakeholders,
I want to organize and track all comments in one place,
so I can ensure nothing gets missed and show progress.

## Job: Onboarding new team members

When a new developer joins my team,
I want to quickly bring them up to speed on our codebase,
so I can get them contributing without extensive hand-holding.
```

### Job Mapping

```
┌─────────────────────────────────────────────────────────────────┐
│ FUNCTIONAL JOBS (What they need to do)                          │
│ • Complete task X                                               │
│ • Find information Y                                            │
│ • Communicate with Z                                            │
├─────────────────────────────────────────────────────────────────┤
│ EMOTIONAL JOBS (How they want to feel)                          │
│ • Feel confident in decisions                                   │
│ • Reduce anxiety about mistakes                                 │
│ • Feel accomplished                                             │
├─────────────────────────────────────────────────────────────────┤
│ SOCIAL JOBS (How they want to be perceived)                     │
│ • Look competent to team                                        │
│ • Be seen as innovative                                         │
│ • Maintain professional reputation                              │
└─────────────────────────────────────────────────────────────────┘
```

---

## Persona Development

Create representative user archetypes.

### Persona Template

```markdown
## [Persona Name]: [Role/Title]

### Demographics
- Role: [Job title]
- Experience: [Years in role]
- Company size: [Team/org context]
- Technical skill: [Low/Medium/High]

### Goals
- Primary: [Main objective]
- Secondary: [Supporting objectives]

### Frustrations
- [Pain point 1]
- [Pain point 2]
- [Pain point 3]

### Behaviors
- Tools currently used: [List]
- Workflow patterns: [Description]
- Decision-making: [How they evaluate options]

### Quote
"[Representative quote from research]"

### Implications for Product
- Must support: [Requirements]
- Should consider: [Nice-to-haves]
- Won't need: [Out of scope]
```

### Persona Pitfalls
- Don't create too many (3-5 max)
- Don't make them fictional without research backing
- Don't include irrelevant demographic details
- Do update them as you learn more

---

## Journey Mapping

Visualize the user experience over time.

### Journey Map Template

```markdown
## Journey: [User goal/task]

### Persona: [Which persona]

| Stage | Awareness | Consideration | Adoption | Usage | Advocacy |
|-------|-----------|---------------|----------|-------|----------|
| **Actions** | [What they do] | | | | |
| **Thoughts** | [What they think] | | | | |
| **Feelings** | [Emotional state] | | | | |
| **Pain Points** | [Frustrations] | | | | |
| **Opportunities** | [How we help] | | | | |

### Key Insights
1. [Insight from journey analysis]
2. [Insight from journey analysis]

### Priority Opportunities
1. [High-impact improvement]
2. [High-impact improvement]
```

---

## Affinity Diagramming

Organize qualitative data into themes.

### Process

1. **Gather** — Collect all observations, quotes, notes
2. **Write** — One insight per sticky note
3. **Cluster** — Group similar items together
4. **Label** — Name each cluster
5. **Prioritize** — Rank by frequency and impact

### Example Output

```markdown
## Research Synthesis: [Topic]

### Theme 1: [Label] (15 mentions)
- "Quote from user A"
- "Quote from user B"
- Observation from session 3
- **Implication:** [What this means for product]

### Theme 2: [Label] (12 mentions)
- "Quote from user C"
- "Quote from user D"
- **Implication:** [What this means for product]

### Theme 3: [Label] (8 mentions)
- ...

### Surprising Findings
- [Unexpected insight that challenges assumptions]

### Gaps in Research
- [What we still don't know]
- [Follow-up research needed]
```

---

## Research Synthesis Template

### Executive Summary

```markdown
## Research Summary: [Study Name]

**Research Goal:** [What we wanted to learn]
**Method:** [Interviews/Surveys/Usability tests/etc.]
**Participants:** [N users, selection criteria]
**Date:** [When conducted]

### Key Findings

1. **[Finding 1]**: [1-2 sentence summary]
   - Impact: [High/Medium/Low]
   - Confidence: [High/Medium/Low]
   - Evidence: [X of Y participants]

2. **[Finding 2]**: [1-2 sentence summary]
   - Impact: [High/Medium/Low]
   - Confidence: [High/Medium/Low]
   - Evidence: [X of Y participants]

### Recommendations

| Priority | Recommendation | Finding | Effort |
|----------|----------------|---------|--------|
| P0 | [Action] | Finding 1 | [T-shirt size] |
| P1 | [Action] | Finding 2 | [T-shirt size] |

### Open Questions
- [What we still need to learn]
```

---

## Quantitative + Qualitative Integration

Combine data types for richer insights.

### Pattern

```
Quantitative: WHAT is happening (metrics, surveys)
Qualitative: WHY it's happening (interviews, observations)
```

### Example

```markdown
## Finding: Onboarding drop-off

### Quantitative Evidence
- 40% of users abandon during step 3 of onboarding
- Average time on step 3: 4.5 minutes (vs 1.2 min for other steps)
- Drop-off correlates with account type (higher for enterprise)

### Qualitative Evidence
- "I didn't understand what information you needed"
- "The form asked for things I didn't have ready"
- "I wanted to explore first before committing details"

### Synthesis
Users abandon step 3 because it requires information they don't
have readily available and feels like commitment before value.

### Recommendation
- Allow skipping optional fields
- Provide preview of product before requiring setup
- Add inline help explaining why each field matters
```

---

## Research Repository

Maintain a searchable archive of research.

### Structure

```
research/
├── studies/
│   ├── 2024-01-user-interviews/
│   │   ├── study-plan.md
│   │   ├── raw-notes/
│   │   ├── synthesis.md
│   │   └── recommendations.md
│   └── 2024-02-usability-test/
├── insights/
│   ├── onboarding.md      # Cumulative insights by topic
│   ├── collaboration.md
│   └── pricing.md
├── personas/
│   ├── developer-dana.md
│   └── manager-marcus.md
└── README.md              # Index and search tips
```

### Tagging System

Tag insights for searchability:
- `#persona:developer`
- `#journey:onboarding`
- `#theme:collaboration`
- `#confidence:high`
- `#impact:high`
