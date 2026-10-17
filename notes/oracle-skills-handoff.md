# Handoff: Oracle Skills for Claude (next session)

> **For Claude:** Read this file first. It summarizes research from a previous session so you don't need to redo it.
> The user writes in Roman Urdu; reply in Roman Urdu unless they switch.

## Context

- User: works with Oracle APEX / Oracle Database. This repo is a fork of `oracle/apex` (APEX samples, GitHub Pages site in `docs/`).
- Goal: make Claude "Oracle-aware" for SQL, PL/SQL, SQLcl, ORDS and APEX work, using Oracle's official agent skills.

## Key links

| What | Link |
|------|------|
| Official skills repo (main source) | https://github.com/oracle/skills |
| DB domain entry point (`db/SKILL.md`) | https://github.com/oracle/skills/blob/main/db/SKILL.md |
| Raw `db/SKILL.md` (for curl) | https://raw.githubusercontent.com/oracle/skills/main/db/SKILL.md |
| Blog: "Route, Don't Flood" (part 2 of 8) | https://blogs.oracle.com/developers/route-dont-flood-using-db-skill-md-to-steer-oracle%e2%80%91aware-assistants-2 |
| Blog: series intro | https://blogs.oracle.com/developers/ai-agents-know-sql-oracle-database-skills-teach-them-oracle |
| Intro mirror (dev.to) | https://dev.to/oracledevs/ai-agents-know-sql-oracle-database-skills-teach-them-oracle-n5m |
| OCI GenAI Agents Quick Start (researched, less relevant) | https://docs.oracle.com/en-us/iaas/Content/generative-ai/get-started-agents.htm |

| **Article 2 full text (dev.to mirror, readable)** | https://dev.to/oracledevs/route-dont-flood-using-dbskillmd-to-steer-oracle-aware-assistants-4el0 |
| Article 3: Controlled Oracle Actions with SQLcl MCP | https://dev.to/oracledevs/controlled-oracle-actions-with-sqlcl-mcp-120e |
| Using Agent Skills to develop with Oracle AI Database | https://dev.to/oracledevs/using-agent-skills-to-develop-with-oracle-ai-database-3h6j |

Note: `blogs.oracle.com` returns HTTP 403 to WebFetch/curl from the cloud container. The same articles (author: Mark Nelson) are on dev.to. To read one in full, use the API: `curl -s https://dev.to/api/articles/3677608 | jq -r .body_markdown` (Article 2 = id 3677608, Article 3 = id 3700591).

## What "Route, Don't Flood" means

- Don't dump all Oracle docs into the model's context ("flood"). That leads to a mix of generic SQL habits and outdated Oracle advice.
- Instead, **route**: `db/SKILL.md` is a front door with a routing table. The assistant reads it, picks the **one** file or folder relevant to the task, reads only that, then acts.
- The routing prompt includes "Do not execute any actions", which keeps discovery separate from tool use. Actual execution comes in a later article about the SQLcl MCP server.
- The series follows an operating model of **route → act → trust**. Article 2 covers "route" only, with no execution.

### Routes by persona (load one file, then decide)

| Persona | Start in | Notes |
|---------|----------|-------|
| App developer | `db/frameworks/`, then `db/appdev/` | Spring, Django, SQLAlchemy: driver, dialect, type mapping, pooling |
| AI engineer | `db/agent/` + `db/features/` | Schema discovery before DML, destructive-op guards, idempotency, Select AI / AI Profiles, Vector Search |
| DBA | `db/performance/`, then `db/security/` | Read the *executed* plan: `DBMS_XPLAN.DISPLAY_CURSOR(NULL, NULL, 'ALLSTATS LAST')` |
| Migration lead | `db/migrations/` + `db/devops/` | Assessment and translation, then delivery (EBR, online ops, testing) |

### Routes by task (follow the order)

- **RAG** (`db/features`): ai-profiles → vector-search → dbms-vector. Set governance first (`OBJECT_LIST`, `SHOW SQL`).
- **Slow query** (`db/performance`): explain-plan → wait-events → optimizer-stats → awr-reports. AWR/ASH may need licensing.
- **Agent-safe schema change** (`db/agent` + `db/migrations`): schema-discovery → destructive-op-guards → idempotency-patterns → schema-migrations
- **MCP setup** (comes later, in "act"): sqlcl-basics → least-privilege skill in `db/security` → sqlcl-mcp-server

### Progressive-discovery prompt (from the article)

```text
Open db/SKILL.md. I'm [persona or task].
Which single skill file should I load next, and why?
Summarize that file in 5 bullets, then propose the next decision (read another file or stop).
Do not execute any actions.
```

Loop: route → read one file → summarize in 5 bullets → decide (stop / read one more / hand off to action).

### Version rules

- The baseline is **Oracle Database 19c** unless a skill says otherwise.
- `DBMS_VECTOR` and most Select AI / AI Profiles examples are documented for **26ai**, and some also work on Autonomous Database. Don't assume these features are on 19c.

### Article 3 preview: SQLcl MCP ("act")

- Run SQLcl as a local MCP server with `sql -mcp` (stdio). Requires SQLcl 25.2+ and Java 17 or 21.
- Connect with a **saved connection name** (`-save -savepwd`). The password never goes into the prompt.
- It starts in the most restrictive "restrict level". Widen it deliberately.
- Session tagging (`MODULE`/`ACTION` in `V$SESSION`) and the MCP activity log let a DBA check what actually ran.
- Use a non-production DB and a read-only, least-privilege user for testing.

## oracle/skills repo summary (verified from README)

- License: UPL-1.0 (free). No OCI account needed.
- Domains:
  - `db/` (active): SQL, PL/SQL, SQLcl, ORDS, VecDB, frameworks, containers, agent-safe workflows.
  - `oci/`: OCI Functions, OKE, IoT, Enterprise AI (GenAI, agents, RAG, APEX integration).
  - `graal/`: GraalVM Native Image.
  - `apex/` and `fusion/`: **placeholders only**, no real content yet.
- `db/` routing highlights:
  - `db/agent/`: safe DML, destructive-operation guards, schema discovery, ORA- error handling
  - `db/features/`: APEX, Select AI, vector search, DBMS_SCHEDULER, AQ, MVs
  - `db/plsql/`, `db/sql-dev/`, `db/performance/`, `db/security/`
  - `db/sqlcl/`: SQLcl, Liquibase, **MCP server** (`db/sqlcl/sqlcl-mcp-server.md`)
  - `db/ords/`: ORDS REST APIs

## How to install

Claude Code (plugin marketplace):

```
/plugin marketplace add oracle/skills
/plugin install db@oracle-skills
/plugin install oci@oracle-skills   # optional
```

Any agent, via the skills CLI:

```bash
npx skills add oracle/skills/db
```

To enable it for this repo for every session, add it to `.claude/settings.json` (`extraKnownMarketplaces` + `enabledPlugins`). **Not done yet. Ask the user first.**

## Earlier finding: OCI GenAI "Skills"

- The `#skills` anchor on the OCI Quick Start page has no real section. Skills are mentioned once ("attached Skills where supported" on explicit containers), with no details.
- That's a paid OCI service for building agents (Responses API, OpenAI SDK). It's less useful for this user than `oracle/skills`.

## Suggested next steps (pending user choice)

1. Install `db@oracle-skills` in Claude Code, and optionally enable it in `.claude/settings.json` for this repo.
2. Look at the next blog in the series about the SQLcl MCP server, so Claude can run queries on the user's DB safely.
3. Help with the user's actual APEX or SQL task using the routed skills.

## Ready-to-paste prompt for a new session

```
Pehle notes/oracle-skills-handoff.md parho (branch ccr-83bc3e15-0wbss2,
repo abdulmajidkhan9439-afk/apex). Phir oracle/skills ka db skill use karke
meri madad karo: <yahan apna kaam likhein>
```
