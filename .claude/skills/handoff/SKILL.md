---
description: Generate a structured handoff document between agents. Use when completing work and passing to the next agent in the workflow.
argument-hint: "[from-agent] to [to-agent]"
---

# Agent Handoff: $ARGUMENTS

Generate a complete handoff document following the protocol in `.claude/rules/team-workflow.md`.

## Handoff Document

```markdown
# Handoff: [From Agent] → [To Agent]
Date: [today]

## Task Owner (Receiving)
[Name of receiving agent and their primary responsibility for this work]

## Context
[Why this task exists. Business goal. What decisions led to this point.]

## Completed Work
[Bulleted list of everything that has been done. Be specific — include file names, API endpoints, schemas, test results as applicable.]

## Pending Work
[Bulleted list of what is NOT done and is being handed over. Be explicit about scope boundaries.]

## Risks & Blockers
[Known issues, assumptions that may break, external dependencies, anything that could slow or block the receiver.]

## Acceptance Criteria
[How the receiver will know their work is complete. Measurable. Maps to user stories if possible.]

## Next Immediate Actions
[The first 2-3 specific actions the receiving agent should take to start work.]

## Questions / Open Items
[Any unresolved questions the receiver may need to answer before or during their work.]
```

Fill in each section based on the current state of the work. Do not leave any section empty — if something doesn't apply, say so explicitly (e.g., "No blockers identified").
