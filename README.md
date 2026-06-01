# emekui/skills

A collection of Claude Code skills for software development workflows.

## Skills

### Karvey Method — Spec-Driven Development

A 13-skill suite implementing the Karvey method: spec-driven development pipeline for enterprise projects.

| Skill | Description |
|-------|-------------|
| `karvey` | Orchestrator — shows pipeline state and next step |
| `karvey-grill` | Pre-spec interrogation (5 branches: problem, users, data, integrations, constraints, stack) |
| `karvey-init` | Initialize spec: create spec.json, proposal.md, Epic in ClickUp or PLAN.md |
| `karvey-requirements` | Generate EARS-format requirements and spec-delta |
| `karvey-mockup` | Generate navigable HTML mockup (3 levels: shell → section → detail) |
| `karvey-design-graphic` | Define visual design system (OKLCH colors, typography, motion) |
| `karvey-architecture` | Enterprise architecture design with explicit security tiers |
| `karvey-tasks` | Generate implementation tasks (10–30 min each, E{n}.F{n}.T{n} naming) |
| `karvey-impl` | Execute tasks in BD→Backend→Frontend order with commits per task |
| `karvey-test` | Run unit and E2E tests, generate test_evidence.md |
| `karvey-qa` | Code review in 6 dimensions: Security, Errors, Consistency, Impact, Env vars, Versioning |
| `karvey-archive` | Merge spec-deltas into living specs, archive change, close Epic |
| `karvey-context` | Dashboard of all active changes and project state |

**Features:**
- Stack-agnostic (detects from package.json, pyproject.toml, go.mod, etc.)
- Management fork: ClickUp or Markdown (PLAN.md)
- Security Tiers 1–4 applied at every phase
- Knowledge graph via graphify at each document-creating phase
- Living specs: cumulative spec documents per capability

**Install all karvey skills:**
```bash
npx skills add emekui/skills@karvey -g
npx skills add emekui/skills@karvey-grill -g
npx skills add emekui/skills@karvey-init -g
npx skills add emekui/skills@karvey-requirements -g
npx skills add emekui/skills@karvey-mockup -g
npx skills add emekui/skills@karvey-design-graphic -g
npx skills add emekui/skills@karvey-architecture -g
npx skills add emekui/skills@karvey-tasks -g
npx skills add emekui/skills@karvey-impl -g
npx skills add emekui/skills@karvey-test -g
npx skills add emekui/skills@karvey-qa -g
npx skills add emekui/skills@karvey-archive -g
npx skills add emekui/skills@karvey-context -g
```

---

### framer-motion-animator-refurbished

Enhanced version of the Framer Motion animator skill with updated package imports (`motion/react`), performance guidelines, and memoization patterns.

```bash
npx skills add emekui/skills@framer-motion-animator-refurbished -g
```

Based on [patricio0312rev/skills@framer-motion-animator](https://skills.sh/patricio0312rev/skills/framer-motion-animator) with improvements from [mindrally/skills@framer-motion](https://skills.sh/mindrally/skills/framer-motion).
