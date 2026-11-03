# agent-skills

Personal fork of agentic-workflow. Tickets and docs are written to a per-project destinations config (Linear or local files; a local docs folder), never to the project's own repo or ticketing system. Run `setup-destinations` once per project.

| Skill | Description |
| --- | --- |
| **diagnosing-bugs** | Diagnosis loop for hard bugs and performance regressions. |
| **domain-modeling** | Build and sharpen a project's domain model (glossary + ADRs). |
| **vallen-code-review** | Review a branch's committed diff with local git only (no PR/remote integration); report goes to the docs root. |
| **grill-me** | Relentless interview to sharpen a plan or design. |
| **grill-with-docs** | Same as grill-me, while writing glossary and ADR docs as you go. |
| **grilling** | Core grilling interview discipline (used by grill-me / grill-with-docs). |
| **setup-destinations** | Configure where tickets and docs get written for a project. |
| **wayfinder**, **to-tickets**, **to-spec** | Plan and publish tickets/specs to the configured destination. |
| **research**, **prototype** | Resolve decision tickets; findings stay in the docs root. |
| **unslop** | Cut AI tells from any writing. |
| **delegate** | Hand specs and follow-ups to subagents: plan, approval gate, implement. Paired with the root `CLAUDE.md`. |

`domain-modeling` is included because `grill-with-docs` depends on it.
