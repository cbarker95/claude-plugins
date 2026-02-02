# PRD Templates

## Purpose

A Product Requirements Document (PRD) defines what you're building and why. Good PRDs:
- Align teams on scope and priorities
- Capture decisions and rationale
- Serve as reference during development
- Enable async collaboration

---

## PRD Structure

### Recommended Template

```markdown
# [Product/Feature Name] PRD

**Version:** 1.0
**Date:** [Date]
**Author:** [Name]
**Status:** Draft | Review | Approved

---

## Executive Summary

[2-3 sentences: What are we building and why does it matter?]

---

## Problem Statement

### The Problem
[What pain or unmet need exists? Be specific.]

### Who Has This Problem
[Target users/personas with this problem]

### Evidence
[Data, research, or quotes demonstrating the problem]

### What Happens If We Don't Solve It
[Consequences of inaction]

---

## Goals & Success Metrics

### Objective
[What outcome are we trying to achieve?]

### Key Results
| Metric | Current | Target | Timeline |
|--------|---------|--------|----------|
| [Metric 1] | [Baseline] | [Goal] | [When] |
| [Metric 2] | [Baseline] | [Goal] | [When] |

### Non-Goals
[What are we explicitly NOT trying to achieve?]

---

## User Stories

### Primary Persona: [Name]

As a [user type],
I want to [action],
so that [benefit].

**Acceptance Criteria:**
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

---

## Scope

### Must Have (P0)
- [Feature/Capability]: [Why essential]
- [Feature/Capability]: [Why essential]

### Should Have (P1)
- [Feature/Capability]: [Value add]
- [Feature/Capability]: [Value add]

### Won't Have (Out of Scope)
- [Feature]: [Why not now]
- [Feature]: [Why not now]

---

## Solution Overview

### Proposed Approach
[High-level description of the solution]

### Key Flows
[Diagram or description of main user flows]

### Technical Considerations
[Architecture, dependencies, constraints]

---

## Design

[Link to Figma/designs or inline mockups]

---

## Open Questions

| Question | Owner | Due Date | Resolution |
|----------|-------|----------|------------|
| [Question 1] | [Name] | [Date] | [Answer] |

---

## Appendix

### Related Documents
- [Link to research]
- [Link to technical spec]
- [Link to designs]

### Changelog
| Date | Version | Changes | Author |
|------|---------|---------|--------|
| [Date] | 1.0 | Initial draft | [Name] |
```

---

## Feature Specification Template

For detailed feature specs (linked from PRD):

```markdown
# Feature Spec: [Feature Name]

**PRD:** [Link to parent PRD]
**Status:** Draft | In Development | Shipped

---

## Overview
[What does this feature do?]

## User Story
As a [user], I want [capability] so that [benefit].

## Detailed Requirements

### Functional Requirements
| ID | Requirement | Priority |
|----|-------------|----------|
| FR-001 | [Requirement] | P0 |
| FR-002 | [Requirement] | P1 |

### Non-Functional Requirements
| ID | Requirement | Criteria |
|----|-------------|----------|
| NFR-001 | Performance | [Metric] |
| NFR-002 | Accessibility | WCAG 2.1 AA |

## UI/UX Specifications
[Detailed interaction specs, edge cases, error states]

## API Specifications
[Endpoints, payloads, responses if applicable]

## Test Scenarios
| Scenario | Steps | Expected Result |
|----------|-------|-----------------|
| Happy path | [Steps] | [Result] |
| Error case | [Steps] | [Result] |

## Launch Checklist
- [ ] Feature flag created
- [ ] Analytics instrumented
- [ ] Documentation updated
- [ ] Support team briefed
```

---

## Lightweight PRD (1-Pager)

For smaller features or quick alignment:

```markdown
# [Feature] — 1-Page PRD

## What
[2-3 sentences describing the feature]

## Why
[Problem being solved, evidence it matters]

## Success
[1-2 measurable outcomes]

## Scope
**In:** [What's included]
**Out:** [What's not included]

## Approach
[High-level solution]

## Timeline
[Target delivery]

## Risks
[Main risks and mitigations]
```

---

## PRD Best Practices

### Do
- Start with the problem, not the solution
- Include explicit "Won't Have" section
- Define measurable success criteria
- Link to research and evidence
- Keep it scannable (use tables, bullets)
- Update as decisions are made

### Don't
- Write a novel (2-5 pages ideal)
- Include implementation details (that's for tech spec)
- Make everything P0
- Leave open questions unassigned
- Treat it as set in stone (iterate!)

---

## Success Metrics Patterns

### Engagement Metrics
```markdown
| Metric | Definition | Target |
|--------|------------|--------|
| DAU/MAU | Daily/Monthly active users | X |
| Retention D1/D7/D30 | % returning after N days | X% |
| Session duration | Average time per session | X min |
| Feature adoption | % users using feature | X% |
```

### Task Completion Metrics
```markdown
| Metric | Definition | Target |
|--------|------------|--------|
| Task success rate | % completing task | >X% |
| Time to complete | Average task duration | <X min |
| Error rate | % encountering errors | <X% |
| Abandonment rate | % starting but not finishing | <X% |
```

### Business Metrics
```markdown
| Metric | Definition | Target |
|--------|------------|--------|
| Conversion rate | % converting to paid | X% |
| Revenue per user | Average revenue | $X |
| Churn rate | % canceling per period | <X% |
| NPS | Net Promoter Score | >X |
```

### Quality Metrics
```markdown
| Metric | Definition | Target |
|--------|------------|--------|
| Bug reports | Issues per release | <X |
| Support tickets | Tickets per feature | <X |
| Performance | Page load time | <X sec |
| Uptime | Availability | >X% |
```

---

## Stakeholder Communication

### For Engineers
Include:
- Clear acceptance criteria
- Edge cases and error handling
- Non-functional requirements
- Technical constraints known

### For Designers
Include:
- User research and personas
- User flows and journeys
- Success metrics for UX
- Accessibility requirements

### For Executives
Include:
- Business impact and metrics
- Strategic alignment
- Resource requirements
- Timeline and milestones

### For Sales/Support
Include:
- Feature positioning
- Common questions (FAQ)
- What's NOT included
- Timeline for GA
