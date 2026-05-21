---
name: cdata-driver-cli
description: Use when user wants to connect, query, explore schema, run SQL, or install drivers for a data source via CLI. Do NOT use for MCP, vendor SDKs, REST APIs. Prereq for cdata- skills.
---

# SKILL: CData Driver CLI

## Data Model

The driver exposes the data source as a relational model:

- Tables that represent queriable entities
- Columns that make up the attributes of tables
- Procedures that support performing actions

---

## CLI Invocation

The CLI installs as `cdatacli` and is on `PATH` after install:

```bash
cdatacli <group> <subcommand> [options]
```

Requires Java 17+. Drivers are discovered from `./` or `./lib/` relative to the CLI executable.

If `cdatacli --version` is missing, install:

- Windows (PowerShell): `iwr https://cdn.cdata.com/cli/install.ps1 | iex`
- macOS/Linux: `curl -fsSL https://cdn.cdata.com/cli/install.sh | bash`

---

## Output Format

All commands return JSON. Pretty-printed by default; append `--compact` for single-line JSON (useful for piping to `jq`). Errors return `{"error": "..."}`.

---

## Command Reference

| Goal | Command |
|---|---|
| List installed drivers | `drivers list` |
| Activate (trial) | `drivers activate --name <Driver> --email E --trial` |
| Activate (key) | `drivers activate --name <Driver> --email E --key KEY` |
| Inspect connection properties | `drivers connectionprops --name <Driver>` |
| Generate driver-specific skill | `drivers skill <Driver>` |
| Create connection | `connection create --driver <Driver> --name N --connectionstring CS` |
| List connections | `connection list` |
| Delete connection | `connection delete --name N` |
| List catalogs | `metadata catalogs --connection N` |
| List schemas | `metadata schemas --connection N` |
| List tables | `metadata tables --connection N` |
| Get columns | `metadata columns --connection N --table T` |
| List procedures | `metadata procedures --connection N` |
| Procedure params | `metadata parameters --connection N --procedure P` |
| Run SQL (SELECT/DML) | `query sql --connection N --sql "SQL"` |
| Execute procedure | `query sql --connection N --sql "EXEC Proc Param='val'"` |

Every subcommand supports `--help` — returns purpose, required args, optional args, and flags. Use when Command Reference doesn't cover your case.

---

## Source-Specific Skills

For popular sources, CData drivers ship source-specific instructions (connection patterns, schema notes, query examples). Generate a ready-to-use skill with:

```bash
cdatacli drivers skill <Driver>
```

This prints a YAML skill prefix followed by the contents of the driver's `sys_instructions` table. Redirect to `~/skills/cdata-<source>/SKILL.md` and load the new skill for future work on that source:

```bash
mkdir -p ~/skills/cdata-<source>
cdatacli drivers skill <Driver> > ~/skills/cdata-<source>/SKILL.md
```

If the driver has no checked-in instructions, the command returns `No instructions available for <driver>` — in that case, proceed with the generic workflow below.

---

## Best Practices

1. **Get columns before querying** — use exact names from schema discovery.
2. **LIMIT first** — sample before full queries.
3. **[Bracket] table and column names** — handles reserved words and spaces.
4. **Filter before joining** — apply WHERE first, add JOINs one at a time.
5. **Date filters prevent timeouts** — large sources time out on unfiltered queries.
6. **Check procedures before concluding impossible** — file operations, bulk exports, auth flows are often stored procedures.
7. **Enum/picklist fields** — look for a dedicated table (e.g. `PickListValues`) before filtering. Don't guess values.

---

## Recommended Workflow

### Step 1: Understand the Goal

Confirm the **source** and **goal** (what to query or accomplish) before proceeding.

If a source-specific SKILL is already installed at `~/skills/cdata-<source>/`, invoke it. Otherwise try `cdatacli drivers skill <Driver>` — if it returns content, save it as a new skill and invoke it. If it returns `No instructions available for <driver>`, continue with the generic workflow.

---

### Step 2: Verify Driver is Available and Activated

```bash
cdatacli drivers list
```

If the driver appears with `"activated": true`, skip to Step 3. If the driver is missing, place its CData JDBC JAR in `./` or `./lib/` next to the CLI executable, then re-run `drivers list` to confirm discovery.

#### Activate

Ask the user for their license key if they have one. Otherwise use `--trial` for a 30-day trial.

```bash
cdatacli drivers activate --name "<Driver>" --email "you@example.com" --trial
cdatacli drivers activate --name "<Driver>" --email "you@example.com" --key "XXXXX-XXXXX"
```

---

### Step 3: Check Connection Properties

If a saved connection already exists (`connection list`), confirm with the user they would like to use it. Otherwise, inspect the driver's connection properties to determine what credentials to ask for:

```bash
cdatacli drivers connectionprops --name "<Driver>"
```

Focus on properties in the Authentication and OAuth categories. Key properties across all sources:
- `AuthScheme` — OAuth, Basic, etc.
- `InitiateOAuth` — OFF, GETANDREFRESH, REFRESH
- `OAuthClientId` / `OAuthClientSecret` — for custom OAuth apps
- `OAuthSettingsLocation` — where OAuth tokens are cached

---

### Step 4: Create a Saved Connection

Confirm the connection string values with the user, then save:

```bash
cdatacli connection create --driver "<Driver>" --name "<connection-name>" \
  --connectionstring "<properties>"
```

Common patterns:

| Auth Type | Connection String |
|---|---|
| OAuth (browser flow) | `AuthScheme=OAuth;InitiateOAuth=GETANDREFRESH` |
| Basic (user/pass) | `AuthScheme=Basic;User=you@example.com;Password=pass` |
| API Token | `User=you@example.com;APIToken=yourtoken` |
| Read-only | Append `ReadOnly=true` to any connection string |

The first query after creating an OAuth connection opens a browser for authentication. Subsequent queries auto-refresh tokens.

```bash
cdatacli connection list
cdatacli connection delete --name "<connection-name>"
```

Connections are saved as encrypted `.conn` files (AES-256):

| OS | Location |
|---|---|
| Windows | `%APPDATA%\CData\<Driver Name> Data Provider\` |
| macOS | `~/Library/Application Support/CData/<Driver Name> Data Provider/` |
| Linux | `~/.config/CData/<Driver Name> Data Provider/` |

---

### Step 5: Discover Schema

```bash
cdatacli metadata tables --connection <name>
cdatacli query sql --connection <name> --sql "SELECT TableName, TableType, Description FROM sys_tables WHERE TableName LIKE '%<keyword>%'"
cdatacli metadata columns --connection <name> --table <TableName>
cdatacli metadata procedures --connection <name>
cdatacli metadata parameters --connection <name> --procedure <ProcedureName>
cdatacli metadata catalogs --connection <name>
cdatacli metadata schemas --connection <name>
```

`metadata tables` and `metadata schemas` accept optional `--catalog` / `--schema` filters.

---

### Step 6: Execute Queries

Use only column names confirmed by previous steps. Build incrementally.

#### SELECT

```bash
cdatacli query sql --connection <name> --sql "SELECT * FROM [TableName] LIMIT 5"
cdatacli query sql --connection <name> --sql "SELECT [Id], [Name], [Status] FROM [TableName] WHERE [Status] = 'Active' LIMIT 10"
cdatacli query sql --connection <name> --sql "SELECT a.[Id], a.[Name], b.[Name] AS Related FROM [TableA] a LEFT JOIN [TableB] b ON a.[ForeignKey] = b.[Id] LIMIT 10"
```

Pipe to `jq` with `--compact`:

```bash
cdatacli query sql --connection <name> --sql "SELECT Id, Name FROM Account" --compact | jq '.[].Name'
```

#### INSERT / UPDATE / DELETE

```bash
cdatacli query sql --connection <name> --sql "INSERT INTO [TableName] (Col1, Col2) VALUES ('val1', 'val2')"
cdatacli query sql --connection <name> --sql "UPDATE [TableName] SET [Col1] = 'new' WHERE [Id] = '123'"
cdatacli query sql --connection <name> --sql "DELETE FROM [TableName] WHERE [Id] = '123'"
```

If writes fail, the connection may have `ReadOnly=true` — recreate without it.

#### Stored Procedures

```bash
cdatacli query sql --connection <name> --sql "EXEC ProcedureName Param1='value1', Param2='value2'"
```

---

### Step 7: Generate Application Code

If the user's goal includes generating application code, use the validated SQL, driver path (from `drivers list`), and connection string to generate standalone code.

#### Driver Locations

**JDBC (Java):**
- Windows: `C:\Program Files\CData\CData JDBC Driver for <DataSource> <Year>\lib\cdata.jdbc.<datasource>.jar`
- macOS: `/Applications/CData/CData JDBC Driver for <DataSource> <Year>/lib/cdata.jdbc.<datasource>.jar`
- CLI-bundled: `./` or `./lib/cdata.jdbc.<datasource>.jar` next to the CLI executable
- Driver class: `cdata.jdbc.<source>.<Source>Driver`
- JDBC URL: `jdbc:<source>:<connection-string>`
- License file: same directory, `cdata.jdbc.<datasource>.lic`
