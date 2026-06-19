---
name: team-capacity
description: Calculates team sprint capacity from Jira using team member availability and historical velocity. Queries the active or upcoming sprint, counts assignable team members, applies availability percentages, and returns the expected capacity in story points. Use when the user asks about team capacity, sprint planning, available bandwidth, or how many points the team can commit to in the next sprint.
disable-model-invocation: true
---

# Team Capacity

Calculates expected sprint capacity in story points by combining team availability data with average velocity from Jira via the Atlassian MCP.

## Workflow

### Step 1 — Identify the Jira project and sprint

Ask the user (or infer from context):
- **Project key** (e.g. `PROJ`)
- **Target sprint** (default: the current active sprint or the next planned sprint)
- **Sprint duration in days** (default: **10 working days** for a 2-week sprint)
- **Average velocity** (reuse output from the `team-velocity` skill if already computed, otherwise calculate it — see that skill)

### Step 2 — Retrieve team members and their availability

Ask the user to provide (or confirm) the team roster for the sprint with individual availability:

| Team Member   | Role       | Availability (%) | Notes                      |
|---------------|------------|------------------|----------------------------|
| Alice Martin  | Developer  | 100%             |                            |
| Bob Chen      | Developer  | 80%              | 2 days off                 |
| Carol Smith   | QA         | 100%             |                            |
| Dave Kumar    | Developer  | 60%              | Part-time this sprint      |

If availability is not provided, default to **100%** per person and note the assumption.

### Step 3 — Calculate capacity

Use the average velocity (story points per person per sprint) as the baseline:

```
points_per_person_per_sprint = average_velocity / team_size_at_velocity_baseline
```

Then for each team member:

```
member_capacity = points_per_person_per_sprint × availability_percentage
```

Sum all members:

```
total_capacity = sum(member_capacity for all members)
```

Round to the nearest whole number.

### Step 4 — Account for overhead

Apply a **focus factor** to account for ceremonies, meetings, and non-feature work (default: **80%** unless the user specifies otherwise):

```
adjusted_capacity = total_capacity × focus_factor
```

### Step 5 — Present results

Output a summary table followed by the recommended commit capacity:

```
| Team Member   | Availability | Capacity (pts) |
|---------------|--------------|----------------|
| Alice Martin  | 100%         | 10             |
| Bob Chen      | 80%          | 8              |
| Carol Smith   | 100%         | 10             |
| Dave Kumar    | 60%          | 6              |
|---------------|--------------|----------------|
| Raw total     |              | 34             |
| Focus factor  | 80%          | -7             |
| **Adjusted**  |              | **27**         |
```

Add a one-sentence recommendation, e.g. "The team can confidently commit to ~27 story points this sprint."

## Notes

- If average velocity has not been computed yet, run the `team-velocity` skill first and use its output here.
- If the team roster for the sprint is unknown, prompt the user to provide it before proceeding.
- If the Atlassian MCP is not authenticated, prompt the user to authenticate it in Cursor Settings before proceeding.
- Default focus factor is 80%; honour any explicit value the user provides (e.g. "we have a 70% focus factor").
- Flag any team members who are new to the team, as their individual velocity baseline may differ from the team average.
