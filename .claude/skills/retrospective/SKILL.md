---
description: Run a project retrospective after delivery. Use after every project or sprint to capture learnings and drive continuous improvement.
disable-model-invocation: true
argument-hint: "[project or sprint name]"
---

# Retrospective: $ARGUMENTS

Generate a structured retrospective document. Review what happened, extract learnings, and define concrete action items.

---

## Retrospective Document

```markdown
# Retrospective: [Project/Sprint Name]

**Date**: [today]
**Facilitator**: hiring-manager-orchestrator
**Period Covered**: [start date] → [end date]
**Participants**: [list of agents/people involved]

---

## 1. Project Summary

**What was delivered**:
[Brief description of what was built or shipped]

**Original scope vs actual scope**:
- Planned: [what was planned]
- Delivered: [what was actually delivered]
- Deferred: [what was cut or moved out — and why]

**Timeline**:
- Planned completion: [date]
- Actual completion: [date]
- Variance: [days ahead / on time / days behind] — [brief reason if off]

---

## 2. What Went Well ✅

*List specific practices, decisions, or moments that worked and should be repeated.*

| What | Why it worked | Keep doing? |
|---|---|---|
| [item] | [reason] | Yes |

---

## 3. What Could Be Better ⚠️

*List specific friction points, delays, or quality issues — without blame.*

| What | Root cause | Impact |
|---|---|---|
| [item] | [cause] | [effect on delivery/quality] |

---

## 4. Surprises / Unexpected Events 🔍

*Things that weren't anticipated — could be positive or negative.*

- [Surprise 1 — what happened and what we learned]
- [Surprise 2]

---

## 5. Action Items

*Concrete, assigned, time-bound. No vague "we should improve X."*

| Action | Owner | Target Date | Success Metric |
|---|---|---|---|
| [specific action] | [agent/person] | [date] | [how we'll know it's done] |

---

## 6. Framework / Process Improvements

*Changes to make to CLAUDE.md, rules, skills, agent prompts, or workflows based on this project.*

| File to Change | What to Change | Why |
|---|---|---|
| [file] | [change] | [reason] |

*These should be implemented before the next project.*

---

## 7. Agent Performance Notes

*What each agent did well or where they fell short — feed into agent memory.*

| Agent | Strengths Observed | Gaps Observed |
|---|---|---|
| [agent] | [strengths] | [gaps] |

---

## 8. Metrics

| Metric | Target | Actual | Notes |
|---|---|---|---|
| Delivery on time | Yes | [yes/no] | |
| Scope delivered | 100% | [%] | |
| Bugs found in QA | Low | [count] | |
| Bugs found post-release | 0 | [count] | |
| Estimated vs actual effort | Aligned | [+/- %] | |

---

## 9. One Thing We'd Do Differently

*If we ran this exact project again, what single change would have the most impact?*

[Answer here]

---

## Sign-Off

This retrospective has been reviewed and action items have been assigned.

- hiring-manager-orchestrator: ☐
- [Other stakeholders]: ☐
```
