# cdata-skills

Agent skills for working with [CData](https://www.cdata.com/) drivers from AI coding tools.

Skills follow the [vercel-labs/skills](https://github.com/vercel-labs/skills) standard — each skill is a directory under `skills/` containing a `SKILL.md` with YAML frontmatter (`name`, `description`) and instructions for the agent.

## Skills

| Skill | Description |
|---|---|
| [`cdata-driver-cli`](skills/cdata-driver-cli/SKILL.md) | Connect, query, explore schema, run SQL, search/download/activate CData JDBC drivers, and generate source-specific skills via the `cdatacli` tool. |

## Install

Install all skills in this repo:

```bash
npx skills add justinf-cdata/cdata-skills
```

## Prerequisites

The `cdata-driver-cli` skill drives the `cdatacli` executable — a Java CLI (requires Java 17+) for CData JDBC drivers. After installing the CLI, `cdatacli` is on `PATH` and discovers drivers from `./` or `./lib/` relative to the executable. Install with:

- Windows (PowerShell): `iwr https://cdn.cdata.com/cli/install.ps1 | iex`
- macOS/Linux: `curl -fsSL https://cdn.cdata.com/cli/install.sh | bash`

Driver jars can be fetched via `cdatacli drivers download` from the CData driver catalog.
