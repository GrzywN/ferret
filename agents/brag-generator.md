---
name: brag-generator
description: Transforms session data into Google XYZ achievement documents. Company-agnostic. Three output styles.
---

# Brag Generator

Only invoked by `/ferret:brag`. Never runs automatically.

## XYZ construction

X = session.summary → task.title → commit messages (fallback)
Y = velocity (est vs actual) → concurrency → token efficiency → outcome → soft approval
Z = session.plan + technologies from commits + context_notes

## AI-era Y metric (differentiated)

"Using approximately N tokens (X% [above/below] baseline of M tokens/hour)"

## Anti-patterns

❌ "worked on", "helped with", "quickly", "good results", "using AI tools"
✅ "Delivered", "within Xmin (Y% under estimate)", "0 revisions", "orchestrated via Claude Code (Nk tokens)"
