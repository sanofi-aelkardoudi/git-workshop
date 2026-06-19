---
name: team-velocity
description: Calculates team sprint velocity from Jira using story points. Queries completed sprints, sums story points per sprint, and returns average velocity over the last N sprints. Use when the user asks about team velocity, sprint capacity, throughput, or how many points the team delivers per sprint.
disable-model-invocation: true
---

# Team Velocity

Calculates average team velocity in story points by querying Jira sprint data via the Atlassian MCP.

## Workflow

### Step 1 — Identify the Jira project and board

Ask the user (or infer from context):
- **Project key** (e.g. `PROJ`)
- **Number of past sprints** to include (default: **last 5 closed sprints**)

### Step 2 — Query completed sprints via Jira MCP

Use the Atlassian MCP (`plugin-atlassian-atlassian`) to search for closed sprints.

JQL to fetch completed issues per sprint:

```
project = "PROJECT_KEY"
  AND sprint in closedSprints()
  AND status = Done
ORDER BY sprint DESC
```

To scope to a specific board, add: `AND sprint in boardSprints(BOARD_ID, "closed")`.

Retrieve the `story_points` (or `customfield_10016` / `customfield_10028` — confirm the field name for this workspace) for each issue.

### Step 3 — Calculate velocity

For each of the last N closed sprints:
1. Sum the story points of all **Done** issues in that sprint.
2. Record `sprint_name → points_completed`.

Then compute:

```
average_velocity = sum(points_completed) / number_of_sprints
```

### Step 4 — Present results

Output a concise summary table followed by the average:

```
| Sprint          | Story Points Completed |
|-----------------|------------------------|
| Sprint 23       | 42                     |
| Sprint 22       | 38                     |
| Sprint 21       | 45                     |
| Sprint 20       | 40                     |
| Sprint 19       | 36                     |
|-----------------|------------------------|
| Average (last 5)| **40.2**               |
```

Add a one-sentence observation if there is a clear trend (e.g. "Velocity has been increasing over the last 3 sprints").

## Notes

- If story points are missing on some issues, note the count of unpointed tickets so the user knows the figure may be understated.
- If the Atlassian MCP is not authenticated, prompt the user to authenticate it in Cursor Settings before proceeding.
- Default sprint count is 5; honour any explicit number the user provides (e.g. "last 3 sprints", "last 10 sprints").
