# 05 - MCP for GenAI Apps

MCP (Model Context Protocol) is a standard way for AI systems to connect to tools and data sources.

Think of MCP like "USB-C for AI tools": one standard interface, many integrations.

> **Spec:** [modelcontextprotocol.io](https://modelcontextprotocol.io/) — official docs and SDKs

## Why MCP matters

Without MCP, each tool integration is custom and brittle.

With MCP:

- Tools expose a standard schema
- AI clients discover available tools
- Calls are structured and auditable
- Integrations become reusable across agents/clients

## Core MCP pieces

- **MCP Server**: hosts tools/resources
- **Tool**: callable function with schema
- **Resource**: read-only data exposed by URI
- **Client**: agent/app that discovers and invokes tools

## When to use MCP vs plain REST

| Use MCP when… | Use plain REST when… |
|---------------|----------------------|
| AI agent needs to discover tools dynamically | Fixed set of backend endpoints |
| Multiple clients (Cursor, Claude, custom) share tools | Single app, known API surface |
| You want standardized tool schemas | Simple CRUD, no AI tool-calling |

## How it fits in a RAG app

Use MCP to connect your assistant to:

- Document systems (Confluence, SharePoint, Git)
- Incident dashboards
- CRM/helpdesk
- Internal APIs

Then your agent can fetch fresh context, not just static indexed docs.

## Example use-cases

- "Get latest leave policy from wiki"
- "Check release notes for version X"
- "Create Jira ticket from chat action"

## Security and governance

- Enforce authentication for MCP servers
- Scope tool permissions by role
- Log every tool call with trace IDs
- Deny dangerous tools by default

## Design guidelines

- Keep tool inputs explicit and typed
- Return structured JSON, not unbounded text
- Make tools idempotent where possible
- Add clear error messages for agent recovery

## Frontend developer mental model

- MCP tool schema ~= typed API contract
- Tool discovery ~= dynamic capability registry
- Tool invocation ~= server action call

If you already design robust frontend-backend contracts, MCP will feel natural.

## Common pitfalls

- Tools with vague parameter names
- No access control boundaries
- Missing audit logs for tool calls
- Letting the model call high-risk tools without review

## Practice task

Create one MCP-style internal tool contract:

- Tool name: `searchPolicies`
- Input: `{ query: string, department?: string }`
- Output: `{ results: Array<{ title, snippet, sourceUrl }> }`

Then integrate it into your answer flow before final LLM generation.
