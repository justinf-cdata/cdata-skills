# cdata-skills

Agent skills for working with [CData](https://www.cdata.com/) drivers from AI coding tools.

Skills follow the [vercel-labs/skills](https://github.com/vercel-labs/skills) standard — each skill is a directory under `skills/` containing a `SKILL.md` with YAML frontmatter (`name`, `description`) and instructions for the agent.

## Skills

| Skill | Description |
|---|---|
| [`cdata-driver-cli`](skills/cdata-driver-cli/SKILL.md) | Connect, query, explore schema, run SQL, and install CData JDBC drivers via the `cdata-cli` tool. |

## Install

Install all skills in this repo:

```bash
npx skills install justinf-cdata/cdata-skills
```

## Prerequisites

The `cdata-driver-cli` skill drives the `cdata-cli` tool — a Java CLI for CData JDBC drivers. The skill assumes the CLI jar is available locally and references it as `/path/to/cdata-cli.jar`.
