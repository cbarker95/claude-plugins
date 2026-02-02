# Roadmap Frameworks

## Now / Next / Later

The simplest and most flexible roadmap format. Avoids false precision of dates.

### Structure

```
┌─────────────────────────────────────────────────────────────────┐
│ NOW (Current Focus)          │ NEXT (Up Next)    │ LATER (Future) │
│ Actively building            │ Planned soon      │ On radar       │
│ Committed scope              │ Scoped, not started│ Ideas, research│
├─────────────────────────────────────────────────────────────────┤
│ • Feature A (in progress)   │ • Feature D       │ • Feature G    │
│ • Feature B (in progress)   │ • Feature E       │ • Feature H    │
│ • Feature C (blocked)       │ • Feature F       │ • Integration X│
└─────────────────────────────────────────────────────────────────┘
```

### When to Use
- Early-stage products with high uncertainty
- Teams that struggle with date commitments
- Communicating with stakeholders who don't need exact dates

### Anti-Pattern
Don't let "Later" become a graveyard. Regularly prune it.

---

## RICE Scoring

Quantitative prioritization framework. Score = (Reach × Impact × Confidence) / Effort

### Components

| Factor | Definition | Scale |
|--------|------------|-------|
| **Reach** | How many users affected per quarter? | Number (e.g., 1000 users) |
| **Impact** | How much will it move the metric? | 3=massive, 2=high, 1=medium, 0.5=low, 0.25=minimal |
| **Confidence** | How sure are we about estimates? | 100%=high, 80%=medium, 50%=low |
| **Effort** | Person-weeks to complete | Number (e.g., 4 weeks) |

### Example Calculation

```
Feature: Auto-save drafts
- Reach: 5000 users/quarter
- Impact: 2 (high - reduces lost work)
- Confidence: 80% (validated in research)
- Effort: 2 person-weeks

RICE Score = (5000 × 2 × 0.8) / 2 = 4000
```

### When to Use
- Comparing many feature options
- Need to justify prioritization decisions
- Stakeholders want "objective" ranking

### Anti-Pattern
Don't over-optimize the math. RICE is a conversation starter, not a decision maker.

---

## MoSCoW Prioritization

Categorical prioritization for scope decisions.

### Categories

| Category | Definition | Rule |
|----------|------------|------|
| **Must Have** | Product doesn't work without it | If missing, don't ship |
| **Should Have** | Important but not critical | Include if possible |
| **Could Have** | Nice to have, low priority | Only if time permits |
| **Won't Have** | Explicitly out of scope | Not this release |

### Application

```markdown
## Feature: User Authentication

### Must Have
- Email/password login
- Password reset flow
- Session management

### Should Have
- OAuth (Google, GitHub)
- Remember me functionality
- Login rate limiting

### Could Have
- Magic link login
- 2FA via authenticator app

### Won't Have (This Release)
- SSO/SAML integration
- Hardware key support
- Biometric authentication
```

### When to Use
- Scoping MVP or release
- Negotiating with stakeholders
- Making trade-off decisions

### Anti-Pattern
If everything is "Must Have," nothing is. Limit Must Haves to true blockers.

---

## Kano Model

Categorize features by user satisfaction impact.

### Categories

| Type | If Present | If Absent | Strategy |
|------|------------|-----------|----------|
| **Basic** | Expected, no delight | Dissatisfaction | Must include |
| **Performance** | More is better | Less is worse | Optimize ROI |
| **Delighters** | Unexpected joy | No impact | Strategic investment |

### Examples

```
Email App:
- Basic: Can send/receive email (expected)
- Performance: Fast search (more = better)
- Delighter: Smart reply suggestions (unexpected joy)
```

### When to Use
- Understanding user expectations
- Identifying differentiation opportunities
- Balancing table-stakes vs innovation

---

## OKR Framework

Objectives and Key Results for goal-setting.

### Structure

```markdown
## Objective: [Qualitative, inspiring goal]

### Key Results:
1. [Quantitative metric] from X to Y
2. [Quantitative metric] from X to Y
3. [Quantitative metric] from X to Y
```

### Example

```markdown
## Objective: Make onboarding delightful

### Key Results:
1. Increase day-1 activation rate from 40% to 60%
2. Reduce time-to-first-value from 15 min to 5 min
3. Improve onboarding NPS from 30 to 50
```

### Rules
- 3-5 Key Results per Objective
- Key Results are outcomes, not outputs
- 70% achievement is success (stretch goals)

### Anti-Pattern
"Ship feature X" is not a Key Result. That's an output. What outcome does it drive?

---

## Prioritization Matrix

2×2 matrix for quick decisions.

### Impact vs Effort

```
                    HIGH IMPACT
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
    │   QUICK WINS       │   BIG BETS         │
    │   Do first         │   Plan carefully   │
    │                    │                    │
LOW ├────────────────────┼────────────────────┤ HIGH
EFFORT                   │                    EFFORT
    │                    │                    │
    │   FILL-INS         │   MONEY PITS       │
    │   Do if time       │   Avoid            │
    │                    │                    │
    └────────────────────┼────────────────────┘
                         │
                    LOW IMPACT
```

### When to Use
- Quick triage of feature requests
- Team alignment exercises
- Stakeholder communication

---

## Roadmap Communication Tips

### For Executives
- Focus on outcomes and business metrics
- Use Now/Next/Later (avoid date promises)
- Highlight strategic alignment

### For Engineering
- Include technical context and dependencies
- Be specific about scope
- Flag risks and unknowns

### For Customers
- Focus on problems being solved
- Avoid specific dates
- Set expectations about "Later" items

### For Sales
- Highlight competitive differentiation
- Provide talking points, not commitments
- Clear "not building" list to set expectations
