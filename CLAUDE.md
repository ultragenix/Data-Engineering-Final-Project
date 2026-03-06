# Project Configuration — DVF+ La Réunion

## Project Context
DVF+ is a real estate and agricultural land data platform for La Réunion (département 974).
Stack: Python, PostgreSQL/PostGIS, dbt, Kestra, Metabase, Streamlit, Docker Compose.
Target: SAFER Réunion (agricultural land management agency).

## Multi-Agent Workflow

This project uses a multi-agent development workflow with 6 specialized agents.
Always read the relevant coordination files before taking action.

## Coordination Files (ALWAYS read these first)
- `PLAN.md` — Project plan, parts breakdown, acceptance criteria (source of truth)
- `STATE.md` — Live project state, progress, history, alerts
- `CORRECTIONS.md` — Pending corrections backlog
- `docs/REFERENCE.md` — Domain reference (acronyms, SAFER context, data sources)

## Available Agents (in .claude/agents/)
| Agent | Role | When to use |
|-------|------|-------------|
| `architect` | Plan & structure | Project start, plan revision |
| `developer` | Implementation | Code a part from PLAN.md |
| `qa-security` | Test & audit | After dev completes a part |
| `validator` | Integration check | After QA approves a part |
| `tech-lead` | Refactoring | Every 3 parts or on demand |
| `documentalist` | Documentation | After validation or on demand |

## Quick Commands (in .claude/commands/)
| Command | Action |
|---------|--------|
| `/project:plan` | Start or revise the project plan |
| `/project:dev` | Implement the next part |
| `/project:qa` | Audit the latest part |
| `/project:validate` | Final validation check |
| `/project:refactor` | Periodic code cleanup |
| `/project:doc` | Update documentation |
| `/project:status` | View project progress |

## Workflow Sequence
```
/project:plan → /project:dev → /project:qa → /project:validate
     ↑                                |
     └──── (if rejected) ─────────────┘

Every 3 validated parts: /project:refactor → /project:doc
```

## Development Rules
- Always commit after each successful agent step
- Use conventional commits: feat(part-N), fix(part-N), refactor, docs
- Never modify files outside the current part's scope
- Always check STATE.md before starting work
- Maximum 3 correction attempts before escalating to architect

## Code Standards
- Python 3.11+ with type hints everywhere
- SQL: uppercase keywords, lowercase identifiers, CTEs over subqueries
- dbt: staging → intermediate → marts pattern (Kimball dimensional)
- Functions < 25 lines, single responsibility
- Explicit error handling, no bare `except:`
- English variable/function names
- French comments, documentation, and user-facing strings
- All SQL queries parameterized (never string concatenation)
- Docker: one service per container, health checks required

## Data Engineering Conventions
- Projection: EPSG:2975 (RGR92/UTM40S) — NEVER Lambert-93
- Department filter: code_departement = '974' or equivalent
- Date format: ISO 8601 (YYYY-MM-DD)
- Currency: EUR, no cents in aggregations (round to integer)
- dbt naming: stg_[source]__[entity], int_[entity]_[transform], dim_[entity], fct_[entity]
- Environment variables: prefixed by DVF_ (e.g., DVF_DB_HOST, DVF_DEEPSEEK_API_KEY)

## Infrastructure
- VPS OVH: 4 vCores / 8 GB RAM / 75 GB SSD
- All services via Docker Compose
- Nginx reverse proxy + Let's Encrypt SSL
- Domain: dashboard.reunia.re (target)