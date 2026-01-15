# Agent Flywheel Workflow Examples

This document provides comprehensive workflow examples showing how to use agent-flywheel tools together effectively, including project setup, AI-powered task selection, team collaboration, and advanced usage patterns.

## Initial Project Setup

### For new projects

First, clone or create your project repository:

```shell
gh repo clone <owner/repo>            # Clone GitHub project
# OR
glab repo clone <owner/repo>          # Clone GitLab project
# OR
mkdir my-new-project && cd my-new-project && git init  # Create new project
```

Initialize beads issue tracking and configure git branches:

```shell
bd init                               # Initialize beads in current project
git add .                             # Stage the new beads files
git commit -m "Initialize beads issue tracking"
git push origin main                  # Push beads setup to main branch
git branch beads-sync main            # Create dedicated sync branch
git push -u origin beads-sync
bd config set sync.branch beads-sync  # Avoid worktree conflicts
```

### For existing projects

If you're working on a project that already has agent-flywheel setup:

```shell
gh repo clone <owner/repo>            # Clone GitHub project
# OR
glab repo clone <owner/repo>          # Clone GitLab project
cd <project-directory>
git checkout beads-sync               # Switch to beads branch if it exists
bd ready                              # Check for ready tasks
```

### Verify setup

Test that your environment is properly configured:

```shell
acfs doctor                           # Quick health check
bd ready                              # Check beads status
bv                                    # Open kanban view
```

## The Flywheel Loop: Plan Step Explained

The "Plan your work" step uses three complementary commands:

### `bv --robot-triage` - Strategic AI Analysis
Provides AI-powered analysis with sophisticated scoring designed for agents:
- **PageRank centrality** - Dependency graph importance (which tasks unlock the most work)
- **Betweenness centrality** - Impact on workflow flow
- **Staleness analysis** - How long tasks have been waiting
- **Risk assessment** - Cross-repo dependencies, activity churn, complexity
- **Time-to-impact** - How quickly completing this task affects other work
- **Priority weighting** - P0 bugs get immediate prioritization

Returns JSON with reasoning like:
```json
{
  "id": "agent-flywheel-xyz",
  "title": "Implement login form",
  "score": 0.85,
  "reasons": [
    "📊 High centrality in dependency graph",
    "⚡ Unblocks 3 other tasks",
    "🕒 Task has been waiting for optimal timing"
  ],
  "claim_command": "bd update agent-flywheel-xyz --status=in_progress"
}
```

### `bd ready` - Actionable Tasks
Shows tasks that are actually ready to work on (no blockers, dependencies satisfied):
```shell
bd ready                              # All ready tasks
bd ready --assignee=myname           # Ready tasks assigned to you
bd ready --unassigned                # Available for claiming
bd ready --pretty                    # Tree format with status/priority symbols
```

### `bv` - Visual Interface

There are two visual interfaces available:

#### Terminal Interface (Built-in)
The standard `bv` command from beads CLI provides terminal-based task management:
- **Kanban board** - Press 'b' for column view (Open, In Progress, Blocked, Closed)
- **Navigation** - j/k keys, gg/G for jumps, vim-style movement
- **Filtering** - Press 'o' for Open, 'r' for Ready, 'c' for Closed tasks
- **Details** - Select tasks to see full information and history
- **Read-only** - Changes made via `bd update`, not through the UI

#### Web Interface (beads_viewer)
For a modern browser-based interface, install **beads_viewer**:

```bash
# Installation (one-time setup)
pip install beads-viewer
# OR
pipx install beads-viewer
```

```bash
# Usage
bv                    # Same command, opens web UI at localhost:8080
# OR
beads-viewer          # Alternative command

# Features:
# - Drag-and-drop Kanban board
# - Visual dependency graphs
# - Live reload (auto-updates from CLI changes)
# - Insights dashboard with project metrics
# - Better for team collaboration and visual planning
```

**Recommendation**: Use beads_viewer (web interface) for visual planning and team collaboration. The terminal interface is faster for quick checks during CLI workflows.

### Agent-Assisted Planning Workflow
1. **Ask agent to analyze**: `cc "Please run bv --robot-triage and bd ready, then recommend which task I should work on"`
2. **Agent processes**: Runs commands, analyzes JSON output, considers dependencies and priorities
3. **Agent recommends**: Provides human-readable recommendation with reasoning
4. **Human decides**: Claims recommended task or asks for alternative analysis
5. **Use agents to implement**: `ntm attach` to work with agents on the claimed task

## The Official Flywheel Loop

Based on https://agent-flywheel.com/learn/flywheel-loop with agent-assisted planning:

```shell
# 1. Agent-assisted planning
cc "Please run bv --robot-triage and bd ready, then recommend which task I should work on and explain your reasoning"

# 2. Claim recommended task (human decision)
bd update <task-id> --status=in_progress

# 3. Use agents to implement
ntm attach myproject                  # Use agents to implement the claimed task
```

### Team Coordination Enhancement

For team workflows, add sync steps:

```shell
# 0. Sync with team first
bd sync

# 1. Agent-assisted planning
cc "Please analyze the current work priorities considering team context and recommend what I should work on next"

# 2. Claim chosen task
bd update <task-id> --status=in_progress
bd sync -m "Claim task from agent recommendation"

# 3. Use agents to implement
ntm attach myproject                  # Use agents to implement the task

# 4. Close and sync when complete
bd close <task-id>
bd sync -m "Complete task"
```

## Solo Developer Workflow

### Starting a New Feature

```shell
# 1. Sync with remote beads (get latest issues from team)
bd sync

# 2. Create epic and tasks
bd create "Implement user authentication system" -t epic -p 2
bd create "Create login form component" -t task -p 3 --parent="agent-flywheel-xyz"
bd create "Add JWT token validation" -t task -p 3 --parent="agent-flywheel-xyz"
bd create "Implement password reset flow" -t task -p 3 --parent="agent-flywheel-xyz"

# 3. Sync beads changes to team
bd sync -m "Add user authentication epic and tasks"

# 4. Run the Flywheel Loop for first task:

# Agent-assisted planning
cc "Please analyze the user authentication tasks and recommend which one to start with, considering dependencies and priorities"

# Claim the recommended task
bd update agent-flywheel-xyz.1 --status=in_progress

# Use agents to implement
ntm attach auth-project               # Use agents to implement the login form component

# Close when complete
bd close agent-flywheel-xyz.1
```

### Completing a Task

```shell
# 1. Commit your code changes
git add .
git commit -m "Implement login form component with validation

- Add React login form with email/password fields
- Include client-side validation
- Add loading states and error handling

Co-Authored-By: Claude Sonnet 4 <noreply@anthropic.com>"

# 2. Push code to main branch
git push origin main

# 3. Close the beads task
bd close agent-flywheel-xyz.1

# 4. Sync beads changes (task completion) with team
bd sync -m "Complete login form component task"
```

## Team Collaboration Workflow

### Daily Team Sync

```shell
# Morning routine - sync with team's beads changes
bd sync

# Agent-assisted daily planning
cc "Please analyze the current project state, check what teammates are working on, identify any blockers, and recommend what I should focus on today"

# Agent will:
# - Run bv --robot-triage for priority analysis
# - Check bd list --status=in_progress for team workload
# - Check bd list --status=blocked for blockers
# - Provide prioritized recommendations with reasoning

# Visual confirmation (optional)
bv                                    # Visual kanban view
```

### Working with Shared Issues

```shell
# Agent-assisted team coordination
cc "Please analyze the available unclaimed work, consider what teammates are working on, and recommend what I should pick up next"

# Agent will:
# - Run bv --robot-triage for priority analysis
# - Check bd ready --unassigned for available tasks
# - Consider team context and dependencies
# - Provide recommendation with reasoning

# Claim the recommended task
bd update agent-flywheel-abc.5 --status=in_progress
bd sync -m "Claim database migration task based on agent analysis"

# Use agents to implement
ntm attach database-migration         # Use agents to implement the task

# Share progress as needed
bd update agent-flywheel-abc.5 --notes "Database schema designed, starting implementation"
bd sync -m "Update progress on database migration"
```

## Epic and Task Management

### Creating an Epic with Tasks

```shell
# 1. Sync first to get latest state
bd sync

# 2. Create the epic with description
bd create "API Rate Limiting System" -t epic -p 1 \
  --description="Implement comprehensive rate limiting to prevent API abuse."

# 3. Get the epic ID and create child tasks
epic_id=$(bd list --type=epic --json | jq -r '.[-1].id')

bd create "Research rate limiting algorithms" -t task -p 3 --parent="$epic_id"
bd create "Implement Redis rate limiter service" -t task -p 2 --parent="$epic_id"
bd create "Add rate limiting middleware" -t task -p 2 --parent="$epic_id"

# 4. Sync the complete epic with team
bd sync -m "Add API rate limiting epic with tasks"
```

## Agent-Assisted Planning Workflow

The agent-flywheel workflow combines human oversight with agent intelligence:

### The Workflow
1. **Planning Phase** (Agent-assisted, human-decided)
   - Agent analyzes priorities using `bv --robot-triage` and `bd ready`
   - Agent provides human-readable recommendations with reasoning
   - Human makes final decision and claims task
   - Creates explicit audit trail for team coordination

2. **Implementation Phase** (Manual with agent assistance)
   - Human uses `ntm attach` to work with agents on the claimed task
   - Agents provide implementation assistance within the ntm session
   - Human guides and monitors the development process
   - Manual task closure and sync when complete

### The Interface Division
- **Agents** - Analyze JSON data from `bv --robot-triage`, provide recommendations
- **`bd` commands** - Task state management (create, claim, close, sync)
- **`ntm attach`** - Work with agents on task implementation
- **`bv` visual** - Human oversight and confirmation (read-only kanban view)

### Why This Design Works
1. **Human strategic control** - Humans decide what to work on based on agent analysis
2. **Agent intelligence** - Sophisticated analysis of priorities, dependencies, risks
3. **Agent-assisted implementation** - Agents help with coding within ntm sessions
4. **Team coordination** - Clear claiming process prevents work conflicts
5. **Human oversight** - Humans maintain control over implementation and completion

This eliminates manual JSON parsing while ensuring humans maintain control over both prioritization and execution.

## Working with Agents

### Using NTM for Implementation

```shell
# Claim task after agent recommendation
bd update agent-flywheel-xyz.3 --status=in_progress

# Use agents for implementation
ntm attach jwt-task                   # Work with agents on JWT validation middleware

# Agents provide implementation guidance within the ntm session
# Human guides the development process and makes decisions
```

## Quality Assurance

### Pre-Commit Workflow

```shell
# Complete quality check workflow
ubs .                                 # Comprehensive bug scan
bd update current-task --notes="Code review passed UBS scan, ready for commit"

# Ensure beads data is synced
bd sync --flush-only                  # Export any pending beads changes
git add .beads/issues.jsonl           # Include beads in commit
git commit -m "Feature implementation with beads tracking"
```

## Best Practices Summary

### Daily Routine - Agent-Assisted
1. **Start day**: `bd sync` (get team updates)
2. **Plan work**: `cc "Analyze priorities and recommend next task"` (agent-assisted planning)
3. **Claim task**: `bd update task-id --status=in_progress` (human decision)
4. **Implement**: `ntm attach project` (work with agents on implementation)
5. **Complete**: `bd close task-id` and `bd sync` (manual completion)

### Team Coordination
- **Sync frequently**: After claiming tasks, during progress updates, and at task completion
- **Use agent analysis**: Let agents parse complex priority data and provide recommendations
- **Claim deliberately**: Human decision-making on task selection prevents conflicts
- **Work with agents**: Use `ntm attach` to collaborate with agents on implementation
- **Update task notes**: Share context and blockers with the team

### Quality Assurance
- **Run UBS before commits**: Catch issues early
- **Work iteratively with agents**: Use `ntm attach` for continuous improvement
- **Document decisions in beads**: Task notes preserve team knowledge
- **Sync beads data with code**: Keep tracking in sync with implementation

This workflow creates an effective development cycle combining AI-guided prioritization with human strategic control and agent-assisted implementation.