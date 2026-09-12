# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Project

**Vibe Coding Demo — Claude + Google Cloud across the SDLC**

A hands-on workshop (presented by Ivan Nardini) demonstrating how Claude integrates with
Google Cloud to streamline the full software development lifecycle. Participants adopt five
professional roles in sequence and use Claude's coding agents, MCP servers, and specialized
developer skills to build, deploy, secure, and analyze an application in an enterprise setting.

The point of the demo is **the handoff**: each role consumes a concrete artifact from the
previous role and produces a concrete artifact for the next one. Keep artifacts on disk, not
in conversation scrollback.

## Before the project starts: confirm the tech stack

**Ask the user for the tech stack before any stage begins. Do not assume one.**

This is a blocking gate. The stack shapes every downstream artifact — the designer's component
conventions, the engineer's architecture, the security engineer's threat surface, and the
analyst's telemetry schema — so guessing it wrong invalidates work across all five roles.

Ask about, at minimum:

- **Frontend** — framework, styling approach, component library (if any)
- **Backend** — language and framework, or "none / serverless only"
- **Google Cloud services** — compute (Cloud Run, GKE, App Engine, Cloud Functions), data
  (Firestore, Cloud SQL, BigQuery), and any Vertex AI usage
- **IaC tool** — Terraform, gcloud CLI scripts, or Cloud Deployment Manager
- **Analytics path** — how events reach BigQuery
- **Target project / region** — the GCP project id and region for the demo

Ask as a single grouped set of questions with recommended defaults, not one at a time. Once the
user answers, record the stack in `artifacts/00-stack.md` and treat that file as the source of
truth every role reads at the start of its stage.

If the user says "you pick", choose a stack, write it to `artifacts/00-stack.md`, state the
choice and the reasoning, and proceed — but still ask first.

## Roles

Five roles, each owning one stage of the lifecycle.

| # | Role | Consumes | Produces | Output location |
|---|------|----------|----------|-----------------|
| 1 | Product Manager | Business goal / idea prompt | PRD, user stories, wireframe or clickable prototype | `artifacts/01-pm/` |
| 2 | UI/UX Designer | PM prototype + design standards | Production-ready UI spec, design tokens, component set | `artifacts/02-design/` |
| 3 | Software Engineer | Front-end designs | Application code, IaC, deployed service on Google Cloud | `src/`, `infra/`, `artifacts/03-engineering/` |
| 4 | Security Engineer | Running app + code + config | OWASP/compliance review, findings, remediations | `artifacts/04-security/` |
| 5 | Growth Manager / Data Analyst | Deployed app + BigQuery telemetry | Usage analysis, insights, feedback to the PM | `artifacts/05-growth/` |

### 1. Product Manager (PM)
Generates ideas for new features or products and turns them into something tangible.
- Frame the problem, target user, and success metrics **before** proposing a solution.
- Produce a PRD with explicit scope boundaries and non-goals.
- Generate initial wireframes or a low-fidelity prototype — rough is correct at this stage.
- Every feature must name the metric it moves; that metric is what role 5 measures later.

### 2. UI/UX Designer
Takes the PM's prototype and designs a solid, production-ready interface.
- Work from design standards / documentation (a design system, brand guide, or platform HIG)
  rather than inventing conventions per screen.
- Deliver design tokens (color, type, spacing) and a component inventory, not just screenshots.
- Cover states the prototype skipped: empty, loading, error, and responsive breakpoints.
- Meet WCAG AA contrast. Accessibility is part of the deliverable, not a later audit.

### 3. Software Engineer
Receives the front-end designs and handles core development and architectural implementation.
- Use MCP servers for live access to Google Cloud and project systems instead of guessing state.
- Use Google Cloud skills for architecture, IaC, and deployment.
- Implement the design tokens as real tokens; do not hardcode values the designer parameterized.
- Infrastructure is code — deployments must be reproducible from `infra/`, never console clicks.
- Instrument the app to emit the events role 5 will query in BigQuery. Wire this in now.

### 4. Security Engineer
Reviews the application for security, compliance, and correct configuration before release.
- Review against OWASP Top 10 and the relevant compliance standard for the demo.
- Check IAM: least privilege on service accounts, no broad primitive roles.
- Check secrets handling (Secret Manager, never in source or env files committed to the repo),
  network exposure, and public access on storage buckets.
- Report findings with severity, a concrete failure scenario, and a specific remediation.
- Findings are blocking input to the engineer, not advisory commentary.

### 5. Growth Manager / Data Analyst
Monitors performance, analyzes user data, and closes the loop back to the PM.
- Query telemetry collected in BigQuery; report against the metrics the PM defined in stage 1.
- Separate what the data shows from what it implies — label inference as inference.
- State sample size and time window on every claim. No conclusions from a day of traffic.
- Deliver a prioritized feedback list that becomes the next PRD's input.

## Execution modes

The workshop runs roles in two ways, and the mode is part of what is being demonstrated.

### Sequential (the default pipeline)
PM → Designer → Engineer → Security → Growth → back to PM.

Each stage starts by reading the previous stage's artifact from disk. Do not begin a stage
before its input artifact exists. The loop from Growth back to PM is what makes this a
lifecycle rather than a one-shot build — run it at least twice when time allows.

### Parallel (independent agents)
Some work fans out. Launch roles as independent agents **only when their inputs do not
depend on each other**, then reconcile the outputs:

- Security review and Growth instrumentation design can run alongside engineering.
- Multiple design directions can be explored concurrently and compared.
- Independent feature tracks can each run their own PM → Design → Eng chain.

Rules for parallel runs:
- Give each agent its own artifact subdirectory to avoid write collisions.
- An agent's report is not shown to the user — relay what matters after it returns.
- Never fabricate a pending agent's result. If it hasn't reported, say it's still running.
- Reconcile conflicts explicitly at the join point; do not let the last writer silently win.

## Repository layout

```
artifacts/          Role handoff outputs — the spine of the demo
  00-stack.md       Confirmed tech stack — written before stage 1, read by every role
  01-pm/            PRD, user stories, wireframes
  02-design/        UI spec, tokens, components
  03-engineering/   ADRs, deployment notes
  04-security/      Findings and remediation log
  05-growth/        Queries, analysis, feedback to PM
src/                Application code
infra/              Infrastructure as code
```

## Conventions

- **Artifacts are files.** A handoff that exists only in conversation cannot be consumed by
  the next role. Write it to `artifacts/<stage>/` with a descriptive name.
- **Stay in role.** When operating as one role, produce that role's deliverable. Note issues
  belonging to other roles in the artifact rather than fixing them out of turn.
- **Name the handoff.** End each stage by stating what was produced, where it lives, and what
  the next role should do with it.
- **Google Cloud state comes from MCP**, not from memory or assumption. Query it.
- **No credentials in the repo.** Secrets go to Secret Manager. This is a demo that gets shared.
- **Cost awareness.** Demo resources are throwaway — prefer scale-to-zero services and tear
  down what the workshop spins up.

## Operating notes for Claude

- Confirm the tech stack with the user before stage 1, and record it in `artifacts/00-stack.md`.
- Check for a relevant skill before falling back to a default approach; this repo's work maps
  onto product, design, Google Cloud, security, and data-analysis skills.
- Confirm before deploying, destroying cloud resources, or anything outward-facing.
- Report outcomes faithfully — if a deploy fails or a stage was skipped, say so plainly.
