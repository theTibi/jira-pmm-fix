# JIRA access for PMM tickets

- Tracker: [perconadev.atlassian.net](https://perconadev.atlassian.net), project **PMM**, keys **PMM-####**.

## With Atlassian MCP (if enabled in Cursor)

1. List MCP tools and read the schema for the JIRA issue tool (name varies by server).
2. Fetch the issue by key (e.g. `PMM-14971`).
3. Prefer fields: summary, description, status, priority, components, `versions` / fix versions, attachments, comment bodies (recent first).

## Without MCP

Ask the user to paste:

- Summary and full description (including steps / expected / actual).
- Affects version and environment.
- Component(s) and labels.
- Stack traces or log excerpts from attachments.

Do not invent ticket details.
