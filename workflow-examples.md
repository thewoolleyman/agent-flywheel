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

### `bv` - Visual Interface (Read-Only)
The standard `bv` command provides visual task management:
- **Kanban board** - Press 'b' for column view (Open, In Progress, Blocked, Closed)
- **Navigation** - j/k keys, gg/G for jumps, vim-style movement
- **Filtering** - Press 'o' for Open, 'r' for Ready, 'c' for Closed tasks
- **Details** - Select tasks to see full information and history

**Note**: The visual interface is read-only. Status changes require CLI commands.

### Combined Planning Workflow
1. **`bv --robot-triage`** - Understand WHY tasks are prioritized (AI analysis for agents)
2. **`bd ready`** - See WHICH tasks you can actually start (no blockers)
3. **`bv`** - Visualize tasks in kanban board for context
4. **Choose intelligently** based on AI insights and current availability

## The Official Flywheel Loop

Based on https://agent-flywheel.com/learn/flywheel-loop:

```shell
# 1. Plan your work
bv --robot-triage                     # AI analysis with reasoning
bd ready                              # See what's ready to work on

# 2. Claim chosen task (manual step - intentional design)
bd update <task-id> --status=in_progress

# 3. Start your agents
ntm spawn myproject --cc=2 --cod=1

# 4. Set context
cm context "Implementing user authentication" --json

# 5. Send initial prompt
ntm send myproject "Let's implement user authentication.
Here's the context: [paste cm output]"

# 6. Monitor and guide
ntm attach myproject

# 7. Scan before committing
ubs .

# 8. Update memory
cm reflect

# 9. Close the task
bd close <task-id>
```

### Team Coordination Enhancement

For team workflows, add sync steps:

```shell
# 0. Sync with team first
bd sync

# 1. Plan your work
bv --robot-triage                     # AI analysis shows optimal work
bd ready                              # See actionable tasks

# 2. Claim chosen task
bd update <task-id> --status=in_progress
bd sync -m "Claim task from AI triage"

# 3-9. [Follow standard flywheel loop steps above]

# 10. Sync completion with team
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

# Plan your work
bv --robot-triage                     # AI analysis prioritizes tasks
bd ready                              # See ready tasks including new auth tasks

# Claim the task
bd update agent-flywheel-xyz.1 --status=in_progress

# Start your agents
ntm spawn auth-project --cc=2 --cod=1

# Set context
cm context "user authentication React login forms" --json

# Send initial prompt
ntm send auth-project "Let's implement the login form component.
Here's the context: [paste cm output]"

# Monitor and guide
ntm attach auth-project

# Scan before committing
ubs .

# Update memory
cm reflect

# Close the task
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

# 4. Update memory with what you learned
cm reflect

# 5. Sync beads changes (task completion) with team
bd sync -m "Complete login form component task"
```

## Team Collaboration Workflow

### Daily Team Sync

```shell
# Morning routine - sync with team's beads changes
bd sync

# Get AI analysis of current project state
bv --robot-triage                     # See what AI recommends for today

# Check team workload and blockers
bd list --status=in_progress          # See what teammates are working on
bd list --status=blocked              # Check for blockers to clear
bv                                    # Visual kanban view
```

### Working with Shared Issues

```shell
# Get AI recommendation from available work
bv --robot-triage                     # Full analysis shows optimal work
bd ready --unassigned                 # See unclaimed actionable tasks

# Claim the recommended task
bd update agent-flywheel-abc.5 --status=in_progress
bd sync -m "Claim database migration task"

# Share intermediate progress
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

## Understanding Manual Task Claiming

The manual `bd update <task-id> --status=in_progress` step is **intentional design** in the beads workflow:

### Why Manual Claiming Is Correct
1. **Deliberate workflow** - Forces conscious decision-making about what to work on
2. **Team coordination** - Creates explicit audit trail of who is working on what
3. **Prevents conflicts** - Avoids multiple people/agents accidentally working on same task
4. **AI-assisted, human-decided** - AI provides intelligence through triage, humans make final choice

### The Interface Division
- **`bv --robot-triage`** - AI analysis and JSON output (designed for agent consumption)
- **`bv`** - Human-friendly visual interface (read-only kanban/navigation)
- **`bd` commands** - Explicit workflow enforcement (create, update, close, sync)

This approach ensures both AI agents and humans can work effectively with beads while maintaining proper workflow discipline.

## Memory and Context Management

### Building Context for Tasks

```shell
# Start task with context from memory
bd update agent-flywheel-xyz.3 --status=in_progress
cm context "authentication JWT tokens React" --json

# Start agents with rich context
ntm spawn jwt-task --cc=2
ntm send jwt-task "Implement JWT token validation middleware. Context: $(cm context 'JWT validation patterns' --json)"

# Update memory as you learn
cm reflect                            # Update procedural memory from sessions
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

### Daily Routine - The Flywheel Loop
1. **Start day**: `bd sync` (get team updates)
2. **Plan work**: `bv --robot-triage` + `bd ready` + `bv` (analysis + actionable + visual)
3. **Claim task**: `bd update task-id --status=in_progress` (explicit claiming)
4. **Start agents**: `ntm spawn project --cc=2 --cod=1`
5. **Set context**: `cm context "topic" --json` (use persistent memory)
6. **Send prompt**: Include CM context in your initial prompt to agents
7. **Monitor**: `ntm attach session` (guide agent progress)
8. **Scan**: `ubs .` (quality check before commit)
9. **Reflect**: `cm reflect` (build persistent knowledge)
10. **Close**: `bd close task-id` and `bd sync` (complete cycle)

### Team Coordination
- **Sync frequently**: After claiming tasks, completing work, or major updates
- **Use descriptive commit messages**: Help teammates understand changes
- **Update task notes**: Share context and blockers with the team
- **Use manual claiming**: The explicit workflow prevents conflicts and ensures coordination

### Quality Assurance
- **Run UBS before commits**: Catch issues early
- **Update memory after learning**: Use `cm reflect` to build context
- **Document decisions in beads**: Task notes preserve team knowledge
- **Sync beads data with code**: Keep tracking in sync with implementation

The flywheel loop creates a compounding cycle where each iteration improves the next through accumulated memory, systematic quality checks, and AI-guided prioritization.