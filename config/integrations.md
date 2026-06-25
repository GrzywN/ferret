# Integrations

## Linear (MCP)
Server: https://mcp.linear.app (or via MCP settings)
Fields needed per issue: id, title, estimate, status, blocking[], blockedBy[], url

## Jira (MCP)
Server: https://mcp.atlassian.com/v1/sse
JQL example: `sprint in openSprints() AND status = "To Do" AND assignee = currentUser()`
Fields: key, summary, timeestimate, issuelinks (Blocks / is blocked by)

## Manual mode (no MCP needed)
/ferret:plan will prompt for task list:
```
PROJ-123: Implement auth service (2h)
PROJ-124: Frontend integration (3h) [after: PROJ-123]
PROJ-127: Update docs (1h)
```
