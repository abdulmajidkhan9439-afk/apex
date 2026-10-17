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

Note: `blogs.oracle.com` returned HTTP 403 to WebFetch/curl from the cloud container. Use web search or the GitHub repo instead.

## What "Route, Don't Flood" means

- Don't dump all Oracle docs into the model's context ("flood"). That leads to a mix of generic SQL habits and outdated Oracle advice.
- Instead, **route**: `db/SKILL.md` is a front door with a routing table. The assistant reads it, picks the **one** file or folder relevant to the task, reads only that, then acts.
- The routing prompt includes "Do not execute any actions", which keeps discovery separate from tool use. Actual execution comes in a later article about the SQLcl MCP server.
- Paths by role: DBA → `performance/`, `security/`; app developer → `appdev/`, `frameworks/`; AI engineer → `agent/`, `features/` (Select AI), `vecdb/`.
- RAG example route: Select AI profiles → Vector Search basics → `DBMS_VECTOR`.

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
