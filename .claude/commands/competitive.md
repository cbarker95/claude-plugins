---
allowed-tools: Read, Write, Glob, Grep, Task, AskUserQuestion, WebFetch, WebSearch
description: Run competitive analysis to inform product strategy
---

# Competitive Analysis Command

Analyze competitors to inform product positioning and roadmap priorities.

## Phase 1: Define Scope

Ask user for analysis parameters:

```
"What would you like to analyze?

Competitors:
- Specific competitors (name them)
- Find competitors for me
- Both

Analysis depth:
- Quick overview (30 min)
- Standard analysis (1-2 hours)
- Deep dive (comprehensive)"
```

## Phase 2: Competitor Discovery

If finding competitors:

### Web Research
```
subagent_type: Explore
prompt: "Based on the product (from PRD), identify:
1. Direct competitors (same solution, same market)
2. Indirect competitors (different solution, same problem)
3. Potential future competitors

Search for:
- '[product category] alternatives'
- '[problem] solutions'
- 'best [product type] tools'

Return list with brief descriptions."
```

### Technology Research
```
subagent_type: Bash
prompt: "Analyze similar projects:
- Search GitHub for similar repositories
- Check Product Hunt for similar launches
- Look at BuiltWith for technology choices

Return relevant findings."
```

## Phase 3: Competitive Analysis

For each competitor, gather information:

### Feature Analysis
```markdown
## [Competitor Name]

### Overview
- **Website**: [URL]
- **Founded**: [Year]
- **Funding**: [Amount/Stage]
- **Team Size**: [Estimate]

### Target Market
- **Primary Users**: [Who]
- **Use Cases**: [What problems they solve]
- **Pricing**: [Model and tiers]

### Features
| Feature | They Have | We Have | Notes |
|---------|-----------|---------|-------|
| Feature A | ✅ | ✅ | Parity |
| Feature B | ✅ | ❌ | Gap |
| Feature C | ❌ | ✅ | Advantage |

### Strengths
- [Strength 1]
- [Strength 2]

### Weaknesses
- [Weakness 1]
- [Weakness 2]

### Positioning
[How they position themselves]

### Reviews/Sentiment
[Summary from G2, Capterra, Twitter, etc.]
```

## Phase 4: Comparative Analysis

### Feature Matrix

```markdown
## Feature Comparison Matrix

| Feature | Us | Competitor A | Competitor B | Competitor C |
|---------|-----|--------------|--------------|--------------|
| Core Feature 1 | ✅ | ✅ | ✅ | ⚠️ |
| Core Feature 2 | ✅ | ✅ | ❌ | ✅ |
| Advanced Feature 1 | ❌ | ✅ | ✅ | ❌ |
| Advanced Feature 2 | ⚠️ | ✅ | ⚠️ | ❌ |

Legend: ✅ Full support | ⚠️ Partial | ❌ Missing
```

### Positioning Map

```markdown
## Positioning Map

                    Enterprise
                        │
         [Competitor C] │ [Competitor A]
                        │
    Simple ────────────┼──────────── Complex
                        │
              [Us]      │ [Competitor B]
                        │
                     SMB
```

### Pricing Comparison

```markdown
## Pricing Comparison

| Tier | Us | Competitor A | Competitor B |
|------|-----|--------------|--------------|
| Free | ✅ 10 users | ✅ 5 users | ❌ |
| Pro | $10/user | $15/user | $12/user |
| Enterprise | Custom | $25/user | Custom |
```

## Phase 5: Strategic Insights

### SWOT Analysis

```markdown
## SWOT Analysis

### Strengths (Internal Positives)
- [Strength 1]
- [Strength 2]

### Weaknesses (Internal Negatives)
- [Weakness 1]
- [Weakness 2]

### Opportunities (External Positives)
- [Opportunity 1]
- [Opportunity 2]

### Threats (External Negatives)
- [Threat 1]
- [Threat 2]
```

### Porter's Five Forces

```markdown
## Porter's Five Forces Analysis

### 1. Competitive Rivalry: [High/Medium/Low]
[Analysis of existing competition intensity]

### 2. Threat of New Entrants: [High/Medium/Low]
[Barriers to entry, capital requirements]

### 3. Threat of Substitutes: [High/Medium/Low]
[Alternative solutions to the same problem]

### 4. Buyer Power: [High/Medium/Low]
[Customer switching costs, alternatives]

### 5. Supplier Power: [High/Medium/Low]
[Dependency on platforms, APIs, talent]
```

## Phase 6: Recommendations

### Differentiation Strategy

```markdown
## Differentiation Opportunities

### 1. [Opportunity Name]
- **Gap Identified**: [What competitors don't do well]
- **Our Advantage**: [Why we can do better]
- **Effort**: [S/M/L]
- **Impact**: [High/Medium/Low]

### 2. [Opportunity Name]
...

## Recommended Focus Areas

1. **Double Down On**: [Areas where we're already strong]
2. **Catch Up On**: [Table stakes we're missing]
3. **Differentiate With**: [Unique value propositions]
4. **Avoid**: [Features that don't align with positioning]
```

### Roadmap Implications

```markdown
## Roadmap Recommendations

### Now (High Priority)
- [Feature]: Close gap with [Competitor]
- [Feature]: Strengthen differentiation

### Next (Medium Priority)
- [Feature]: Match table stakes
- [Feature]: New differentiator

### Watch List
- [Feature]: Monitor if competitors add
- [Trend]: Could disrupt market
```

## Phase 7: Output

Save analysis to:

```
docs/product/competitive-analysis/
├── OVERVIEW.md           # Executive summary
├── competitors/
│   ├── competitor-a.md   # Detailed competitor profiles
│   ├── competitor-b.md
│   └── competitor-c.md
├── FEATURE-MATRIX.md     # Comparison matrix
├── SWOT.md              # SWOT analysis
└── RECOMMENDATIONS.md    # Strategic recommendations
```

## Keeping Updated

```markdown
## Competitive Intelligence Schedule

### Monthly
- [ ] Check competitor changelogs/releases
- [ ] Review new features
- [ ] Update feature matrix

### Quarterly
- [ ] Deep dive on positioning changes
- [ ] Pricing updates
- [ ] New competitor scan

### Triggers
- [ ] Competitor funding announcement
- [ ] Major feature launch
- [ ] Pricing change
```

## Options

- `--competitors [names]` — Specific competitors to analyze
- `--depth [quick|standard|deep]` — Analysis depth
- `--focus [features|pricing|positioning]` — Focus area
- `--output [file]` — Output location

## Related Commands

- `/prd` — Update PRD with competitive insights
- `/roadmap` — Inform roadmap with competitive analysis
- `/strategy` — Overall product strategy
