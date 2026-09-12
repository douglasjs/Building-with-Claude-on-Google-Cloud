# Vibe Coding — Claude + Google Cloud v0.1

A workspace for vibecoding initial documents across the software development lifecycle.

Five roles run in sequence, each reading the previous role's artifact from disk and writing its
own for the next one:

PM → Designer → Engineer → Security → Growth → back to PM

## Layout

```
artifacts/
  00-stack.md       Tech stack — written first, read by every role
  01-pm/            PRD, user stories, wireframes
  02-design/        UI spec, tokens, components
  03-engineering/   ADRs, deployment notes
  04-security/      Findings and remediation log
  05-growth/        Queries, analysis, feedback to PM
src/                Application code
infra/              Infrastructure as code
```

Handoffs are files, not chat messages. See `CLAUDE.md` for the full working agreement.
