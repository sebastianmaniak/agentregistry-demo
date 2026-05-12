# MCP Demo Script — Agentregistry Conference Demo

## Prerequisites (before going on stage)

- arctl daemon running: `arctl daemon start`
- Docker running
- kind cluster ready: `kind create cluster --name agentregistry`
- kagent installed on cluster (see kagent OSS quickstart)
- Clean slate: `arctl mcp list` shows no existing servers
- Claude Code installed and open

---

## Act 1: Scaffold the MCP Server (~1 min)

```bash
# Scaffold
arctl mcp init python ops-server
# Description: Customer ops and knowledge base MCP server
# Author: Sebastian
# Email: sebastian.maniak@solo.io

# Show the default tool
cat ops-server/src/tools/echo.py
# "Here's the default echo tool — let's replace it with real ops tools"
```

## Act 2: Add Tools & Build (~1.5 min)

```bash
# Add our tools
arctl mcp add-tool search_docs --project-dir ops-server
arctl mcp add-tool get_ticket --project-dir ops-server
arctl mcp add-tool list_open_tickets --project-dir ops-server

# [Copy in the pre-written tool implementations]
# Show the tool code briefly:
cat ops-server/src/tools/search_docs.py
cat ops-server/src/tools/get_ticket.py

# Show all tools
ls ops-server/src/tools/

# Build and publish to the registry
arctl mcp build ops-server --image ops-server
arctl mcp publish ops-server --type oci --package-id ops-server
```

## Act 3: The Registry (~30s)

```bash
arctl mcp list
```

Then open http://localhost:12121 and show the server in the UI.

## Act 4: Deploy Locally (~30s)

```bash
arctl deployments create sebastianmaniak/ops-server \
  --type mcp \
  --version 0.1.0
```

## Act 5: Deploy to Kubernetes (~1 min)

```bash
# Load image to kind (since we built locally)
kind load docker-image ops-server:latest --name kagent

# Deploy to the kind cluster
arctl deployments create sebastianmaniak/ops-server \
  --type mcp \
  --provider-id kubernetes-default \
  --namespace default \
  --version 0.1.0

# Verify
kubectl get pods -n kagent | grep ops-server
```

## Act 6: The AI Agent (~1 min)

Open Claude Code. Add the MCP server:

```bash
claude mcp add --transport http ops-server http://localhost:21212/mcp
```
claude mcp add --transport http ops-server1 http://172.18.0.3/mcp

### Demo prompts (in order):

1. **"Show me all open ops tickets"**
   Claude calls `list_open_tickets` -> sees TICK-1042 (payment timeout errors), TICK-1057 (502 gateway errors), etc.

2. **"Get the details on ticket TICK-1042"**
   Claude calls `get_ticket` -> full ticket detail with customer context, timestamps, severity

3. **"Search our docs for anything related to payment timeouts"**
   Claude calls `search_docs` -> finds relevant KB articles and troubleshooting guides

4. **"Based on everything you've found, draft a response for the customer on TICK-1042"**
   Claude synthesizes tickets + docs -> writes a customer-ready response with root cause and next steps

### Punchline:

> "From zero to an AI-powered ops agent — scaffolded, registered, deployed, and connected — in under 5 minutes."

---

## Cleanup

```bash
arctl deployments list
arctl deployments delete <ops-deployment-id>
arctl mcp delete ops-server --version 0.1.0
```
