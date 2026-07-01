# Graph Intel

Graph Intel is a code intelligence platform that builds a graph representation of source code, enriches it with domain-specific extractors, stores it in Neo4j, and exposes the graph through an HTTP Query Service and an MCP server for AI assistants.

---

# Clone Graph Intel

```bash
git clone <graph-intel-repository-url>
cd graph-intel
```

Open the cloned repository in your preferred IDE:

- Visual Studio Code
- GoLand
- Zed
- Any Go-compatible editor

Continue with the remaining setup steps from the integrated terminal in your IDE or from a terminal opened in the repository root.

When you first open the project, your IDE may automatically download the Go module dependencies defined in `go.mod`. If it doesn't, run:

```bash
go mod download
```

---

# Prerequisites

- Go 1.24+
- Docker
- Claude Code (for MCP integration)

---

# Install Graphify

## Recommended Installation

```bash
# Install uv (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install Graphify
uv tool install graphifyy
```

> **Note:** The official PyPI package is **`graphifyy`** (double **y**). The CLI command is still **`graphify`**.

If the `graphify` command isn't found afterwards, run:

```bash
uv tool update-shell
```

Restart your terminal.

---

## Platform-specific uv Installation

### macOS / Linux

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

### Windows

```powershell
winget install astral-sh.uv
```

---

## Alternative Install Methods

```bash
pipx install graphifyy
```

or

```bash
pip install graphifyy
```

---

## Verify

```bash
graphify --version
```

or

```bash
graphify --help
```

You should be able to run `graphify` from any terminal before continuing.

---

# Start Neo4j

Choose a password for your local Neo4j instance and use the same password throughout the remaining setup steps.

```bash
docker run -d \
  --name graph-intel-neo4j \
  -p 7474:7474 \
  -p 7687:7687 \
  -e NEO4J_AUTH=neo4j:<password> \
  neo4j:5
```

Neo4j Browser:

```
http://localhost:7474
```

Login with:

```
Username: neo4j
Password: <password>
```

---

# Configure Repositories

Edit:

```
config/repos.yaml
```

Determine the default branch of each repository:

```bash
cd /path/to/repository
git branch --show-current
```

Example output:

```
main
```

or

```
master
```

Use that branch name in `config/repos.yaml`.

Example:

```yaml
repositories:
  - name: my-service
    url: file:///Users/<username>/repos/my-service
    branch: main
```

Each repository should reference an existing local Git clone using `file://`.

---

# Index Repositories

```bash
export NEO4J_URI=neo4j://127.0.0.1:7687
export NEO4J_PASSWORD=<password>

go run ./cmd/indexer \
    -config config/repos.yaml \
    -workdir ./workdir-test
```

A successful run will look similar to:

```
success: 68058 nodes, 154286 links
```

The indexer exits automatically after indexing completes.

---

# Start the Query Service

```bash
export NEO4J_URI=neo4j://127.0.0.1:7687
export NEO4J_PASSWORD=<password>

go run ./cmd/query-service
```

The Query Service listens on:

```
http://localhost:8080
```

Keep this terminal running while using Graph Intel.

---

# Build the MCP Server

```bash
go build -o mcp-server ./cmd/mcp-server
```

---

# Register with Claude Code

```bash
claude mcp add graph-intel \
    -s user \
    env QUERY_SERVICE_URL=http://localhost:8080 \
    /absolute/path/to/mcp-server
```

Restart **Claude Code completely** after registering the MCP server.

---

# Verify the Setup

## Verify Neo4j

Open:

```
http://localhost:7474
```

Run:

```cypher
MATCH (n:Entity)
RETURN count(n);
```

---

## Verify the Query Service

```bash
curl http://localhost:8080/health
```

Replace `<repository-name>` with one of the repositories configured in `config/repos.yaml`.

```bash
curl http://localhost:8080/overview/<repository-name>
```

---

## Verify the MCP

```bash
claude mcp list
```

Expected:

```
graph-intel
✔ Connected
```

---

# Example Claude Prompt

```
Use only the Graph Intel MCP tools.

Onboard me to the repository. Explain the architecture, major modules,
entry points, HTTP APIs, Kafka flows, SQL objects, dependencies, and
the most important components I should understand first.

Do not inspect local files.
```

---

# Troubleshooting

## Neo4j connection refused

Verify the container is running:

```bash
docker ps
```

---

## Query Service won't start

Check whether port 8080 is already in use:

```bash
lsof -i :8080
```

Stop the existing process if necessary.

---

## Verify Query Service

```bash
curl http://localhost:8080/health
```

---

## Verify Repository Overview

```bash
curl http://localhost:8080/overview/<repository-name>
```
