---
description: (Optional) Generate a Google XYZ achievement document from completed sessions. Produces eval summaries, case studies, or blog post drafts.
---

# /ferret:brag – XYZ achievement document (optional)

This command is entirely optional. It generates polished output from your completed
session data. It does not affect tracking or git data.

## Arguments: $ARGUMENTS

- `--style eval` – evaluation summary (default)
- `--style case-study` – technical case study for portfolio
- `--style blog` – blog post draft
- `--tasks PROJ-123,PROJ-124` – specific tasks (default: all in period)
- `--period "March 2026"` / `"Q1 2026"` / `"last-30-days"`

## Instructions

### Step 1 – Load sessions
Read `data/completed/` for the period/task filter.
Cross-reference `.ferret.ndjson` for commit + token data per session.

### Step 2 – Build XYZ per session
```
X = session.summary  (or task title if no summary)
Y = priority: velocity (under/over estimate) → concurrency → token efficiency → outcome
Z = session.plan + tech from commits + tags
```

Strong X verbs: Delivered, Shipped, Implemented, Resolved, Designed

### Step 3 – Style-specific output

**`--style eval`** → `output/brag-docs/YYYY-MM-eval.md`
- One XYZ entry per task, grouped by competency
- Executive summary (top 3-5)
- Growth areas from `needs-revision` outcomes
- Tags: #delivery #quality #velocity #initiative #collaboration (company-agnostic)

**`--style case-study`** → `output/case-studies/YYYY-MM-{slug}.md`
Sections: Problem · Approach · Implementation · Results · Learnings
Include: total time, tokens, concurrency factor, specific technologies

**`--style blog`** → `output/blog-posts/YYYY-MM-{slug}.md`
Narrative, accessible, 800-1500 words
Hook → XYZ "so what" → technical insights from plan/notes → what you'd do differently

### Token efficiency angle (novel for AI-era docs)
Always include: "This work cost approximately N tokens (X% vs baseline)"
This is a differentiating metric in AI-era portfolios.

### Anti-patterns
❌ "worked on" / "helped with" / "quickly" / "good results"
✅ "Shipped", "within 45min (30% under estimate)", "0 post-merge revisions"
