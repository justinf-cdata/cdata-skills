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

## CLI Location

The CLI is a Java jar. Invoke with a path to the jar:

```bash
java -jar "/path/to/cdata-cli.jar" <command> [subcommand] [options]
```

---

## Output Formats

| Flag | Description | Best For |
|---|---|---|
| `-f json` | Structured JSON (default) | AI agents, programmatic use |

---

## Command Reference

| Goal | Command |
|---|---|
| List installed drivers | `drivers list` |
| Download a driver | `drivers download <source>` |
| Activate license (trial) | `drivers activate <source> --trial --name N --email E` |
| Activate license (key) | `drivers activate <source> --key KEY --name N --email E` |
| Connection properties | `drivers connectionproperties <source> -f table` |
| Create connection | `connection create --driver <source> --name N --connectionstring CS` |
| List connections | `connection list` |
| Delete connection | `connection delete --name N` |
| List tables | `query tables --connection N` |
| Get columns | `query columns --connection N --table T` |
| List procedures | `query procedures --connection N` |
| Procedure params | `query parameters --connection N --procedure P` |
| List catalogs | `query catalogs --connection N` |
| List schemas | `query schemas --connection N` |
| Run SELECT | `query sql --connection N --sql "SQL" -f table` |
| Run write | `query sql --connection N --sql "SQL"` |
| Execute procedure | `query sql --connection N --sql "EXEC Proc Param='val'"` |

Every subcommand supports `-h` / `--help` — returns purpose, required args, optional args, and flags. Use when Command Reference
doesn't cover your case:


---

## Source-Specific Skills

For popular sources, CData drivers expose a source-specific guide as a system table (e.g. `sys_guide`) — connection patterns,
schema notes, and query examples tailored to that source. Not all sources will have this system table.

```bash
<cli> query sql --connection <name> --sql "SELECT * FROM sys_guide"
```

Use the returned content to author a cdata-<source> skill at ~/skills/cdata-<source>/SKILL.md following standard skill
conventions — concise triggering description, imperative body, examples from the guide. Load the new skill for future work on that
source.

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

If the source matches a source-specific SKILL, invoke it. If a source-specific SKILL does not exist but sys_guide does, create a skill and invoke it.

---

### Step 2: Install the Driver

```bash
java -jar "/path/to/cdata-cli.jar" drivers list
```

If the driver is found and activated, skip to Step 3.

#### Download

```bash
java -jar "/path/to/cdata-cli.jar" drivers download <source>
java -jar "/path/to/cdata-cli.jar" drivers download <source> --version <version>
```

#### Activate

Ask the user for their license key if they have one. Otherwise use `--trial` for a 30-day trial.

```bash
java -jar "/path/to/cdata-cli.jar" drivers activate <source> --trial --name "Your Name" --email "you@example.com"
java -jar "/path/to/cdata-cli.jar" drivers activate <source> --key "XXXXX-XXXXX" --name "Your Name" --email "you@example.com"
```
---

### Step 3: Check Connection Properties

If a saved connection already exists, confirm with the user they would like to use it, if not query the driver's metadata to determine what credentials to ask for:

```bash
java -jar "/path/to/cdata-cli.jar" drivers connectionproperties <source> -f table
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
java -jar "/path/to/cdata-cli.jar" connection create --driver <source> --name <connection-name> --connectionstring "<properties>"
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
java -jar "/path/to/cdata-cli.jar" connection list
java -jar "/path/to/cdata-cli.jar" connection delete --name <connection-name>
```

Connections are saved to `~/.cdata/connections.json`.

---

### Step 5: Discover Schema

```bash
java -jar "/path/to/cdata-cli.jar" query tables --connection <name> -f table
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "SELECT TableName, TableType, Description FROM sys_tables WHERE TableName LIKE '%<keyword>%'"
java -jar "/path/to/cdata-cli.jar" query columns --connection <name> --table <TableName> -f table
java -jar "/path/to/cdata-cli.jar" query procedures --connection <name> -f table
java -jar "/path/to/cdata-cli.jar" query parameters --connection <name> --procedure <ProcedureName> -f table
java -jar "/path/to/cdata-cli.jar" query catalogs --connection <name>
java -jar "/path/to/cdata-cli.jar" query schemas --connection <name>
```

---

### Step 6: Execute Queries

Use only column names confirmed by previous steps. Build incrementally.

#### SELECT

```bash
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "SELECT * FROM [TableName] LIMIT 5" -f table
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "SELECT [Id], [Name], [Status] FROM [TableName] WHERE [Status] = 'Active' LIMIT 10"
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "SELECT a.[Id], a.[Name], b.[Name] AS Related FROM [TableA] a LEFT JOIN [TableB] b ON a.[ForeignKey] = b.[Id] LIMIT 10"
```

#### INSERT / UPDATE / DELETE

```bash
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "INSERT INTO [TableName] (Col1, Col2) VALUES ('val1', 'val2')"
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "UPDATE [TableName] SET [Col1] = 'new' WHERE [Id] = '123'"
java -jar "/path/to/cdata-cli.jar" query sql --connection <name> --sql "DELETE FROM [TableName] WHERE [Id] = '123'"
```
If writes fail, the connection may have `ReadOnly=true` — recreate without it.

#### Stored Procedures

```bash
java -jar target/cdata-cli.jar query sql --connection <name> --sql "EXEC ProcedureName Param1='value1', Param2='value2'"
```
---

### Step 7: Generate Application Code

If the user's goal includes generating application code, use the validated SQL, driver path (from `drivers list`), and connection string to generate standalone code.

#### Driver Locations

**JDBC (Java):**
- Windows: `C:\Program Files\CData\CData JDBC Driver for <DataSource> <Year>\lib\cdata.jdbc.<datasource>.jar`
- macOS: `/Applications/CData/CData JDBC Driver for <DataSource> <Year>/lib/cdata.jdbc.<datasource>.jar`
- CLI: `./drivers/cdata.jdbc.<datasource>.jar`
- Driver class: `cdata.jdbc.<source>.<Source>Driver`
- JDBC URL: `jdbc:<source>:<connection-string>`
- License file: same directory, `cdata.jdbc.<datasource>.lic`
