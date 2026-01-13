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

### Agent-Assisted Planning Workflow
1. **Ask agent to analyze**: `cc "Please run bv --robot-triage and bd ready, then recommend which task I should work on"`
2. **Agent processes**: Runs commands, analyzes JSON output, considers dependencies and priorities
3. **Agent recommends**: Provides human-readable recommendation with reasoning
4. **Human decides**: Claims recommended task or asks for alternative analysis
5. **Automation takes over**: Background process picks up in-progress tasks automatically

## The Official Flywheel Loop

Based on https://agent-flywheel.com/learn/flywheel-loop with agent-assisted planning:

```shell
# 1. Agent-assisted planning
cc "Please run bv --robot-triage and bd ready, then recommend which task I should work on and explain your reasoning based on the AI analysis"

# 2. Claim recommended task (human decision)
bd update <task-id> --status=in_progress

# 3-9. Automated flywheel loop picks up in-progress tasks
# Background process automatically handles:
# - Starting agents (ntm spawn)
# - Setting context (cm context)
# - Sending prompts
# - Monitoring progress (ntm attach)
# - Quality scanning (ubs)
# - Memory updates (cm reflect)
# - Task completion (bd close)
```

### Manual Flywheel Loop (For Direct Control)

When you want direct control over the process:

```shell
# 1. Plan your work (agent-assisted)
cc "Analyze current work priorities and recommend next task"

# 2. Claim chosen task
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

# 1. Agent-assisted planning
cc "Please analyze the current work priorities considering team context and recommend what I should work on next"

# 2. Claim chosen task
bd update <task-id> --status=in_progress
bd sync -m "Claim task from agent recommendation"

# 3-9. Automated flywheel loop executes
# Background automation handles the full cycle automatically

# 10. Task completion synced automatically
# bd sync happens automatically when automation completes tasks
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

# Automated execution takes over (or manual if you prefer control):
# - Background process automatically handles the full flywheel loop
# - Spawns agents, sets context, monitors, scans, reflects, and closes task

# Manual execution (alternative for direct control):
ntm spawn auth-project --cc=2 --cod=1
cm context "user authentication React login forms" --json
ntm send auth-project "Let's implement the login form component. Context: [paste cm output]"
ntm attach auth-project
ubs .
cm reflect
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

# Automated progress tracking or manual updates
# Automation handles progress and completion automatically
# Or manual progress sharing:
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

## Agent-Assisted Planning + Automated Execution

The agent-flywheel workflow combines human oversight with agent intelligence and automated execution:

### The Two-Phase Model
1. **Planning Phase** (Agent-assisted, human-decided)
   - Agent analyzes priorities using `bv --robot-triage` and `bd ready`
   - Agent provides human-readable recommendations with reasoning
   - Human makes final decision and claims task
   - Creates explicit audit trail for team coordination

2. **Execution Phase** (Fully automated)
   - Background process monitors for in-progress tasks
   - Automatically starts flywheel loop (spawn agents, set context, monitor, scan, reflect)
   - Handles the full development cycle without manual intervention
   - Syncs completion automatically

### The Interface Division
- **Agents** - Analyze JSON data from `bv --robot-triage`, provide recommendations
- **`bd` commands** - Task state management (create, claim, close, sync)
- **Background automation** - Picks up in-progress tasks and executes flywheel loop
- **`bv` visual** - Human oversight and confirmation (read-only kanban view)

### Why This Design Works
1. **Human strategic control** - Humans decide what to work on based on agent analysis
2. **Agent intelligence** - Sophisticated analysis of priorities, dependencies, risks
3. **Automated execution** - Agents handle the repetitive flywheel loop automatically
4. **Team coordination** - Clear claiming process prevents work conflicts
5. **Continuous improvement** - Memory and context build up automatically through cm reflect

This eliminates manual JSON parsing while ensuring humans maintain strategic control over work prioritization.

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

### Daily Routine - Agent-Assisted + Automated
1. **Start day**: `bd sync` (get team updates)
2. **Plan work**: `cc "Analyze priorities and recommend next task"` (agent-assisted planning)
3. **Claim task**: `bd update task-id --status=in_progress` (human decision)
4. **Automated execution**: Background process handles flywheel loop automatically:
   - Spawns agents (`ntm spawn`)
   - Sets context (`cm context`)
   - Sends prompts with context
   - Monitors progress (`ntm attach`)
   - Quality scans (`ubs .`)
   - Updates memory (`cm reflect`)
   - Closes task (`bd close`) and syncs (`bd sync`)

### Team Coordination
- **Sync frequently**: Automated for task completion, manual after claiming tasks
- **Use agent analysis**: Let agents parse complex priority data and provide recommendations
- **Claim deliberately**: Human decision-making on task selection prevents conflicts
- **Monitor automation**: Check automated flywheel loop progress, intervene when needed
- **Update task notes**: Share context and blockers with the team

### When to Use Manual vs Automated
- **Use automated**: Standard feature development, bug fixes, routine work
- **Use manual**: Complex tasks needing human guidance, experimental work, learning new domains
- **Use hybrid**: Agent-assisted planning with selective manual execution steps

### Quality Assurance
- **Run UBS before commits**: Catch issues early
- **Update memory after learning**: Use `cm reflect` to build context
- **Document decisions in beads**: Task notes preserve team knowledge
- **Sync beads data with code**: Keep tracking in sync with implementation

The flywheel loop creates a compounding cycle where each iteration improves the next through accumulated memory, systematic quality checks, and AI-guided prioritization.