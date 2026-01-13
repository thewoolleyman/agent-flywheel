# Agent Flywheel Workflow Examples

This document provides detailed workflow examples showing how to use agent-flywheel tools together effectively, including proper beads issue tracking and team synchronization.

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

# 4. Check what's ready to work on
bd ready

# 5. Start working on first task
bd update agent-flywheel-xyz.1 --status in_progress

# 6. Plan work visually
bv                                    # Check tasks in kanban view

# 7. Start agents for the project
ntm spawn auth-project --cc=2 --cod=1

# 8. Set context from memory
cm context "user authentication React" --json

# 9. Send initial prompt with context
ntm send auth-project "Implement login form component. Context: [paste cm output]"

# 10. Monitor agent work
ntm attach auth-project
```

### During Development

```shell
# Check agent progress
ntm list                              # See active sessions
ntm send auth-project "Show me the current login form code"

# Update task with progress notes
bd update agent-flywheel-xyz.1 --notes "Login form component created, need to add validation"

# Run quality checks before committing code
ubs .                                 # Scan for bugs and issues
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

# Check what your teammates are working on
bd list --assignee="teammate-name"
bv                                    # Visual kanban view

# Check for conflicts or blockers
bd list --status=blocked
```

### Working with Shared Issues

```shell
# Claim an available task
bd ready                              # See unassigned tasks
bd update agent-flywheel-abc.5 --assignee="myname" --status=in_progress

# Sync your claim with the team immediately
bd sync -m "Claim database migration task"

# Work on the task...
# [development work happens here]

# Share intermediate progress without committing code
bd sync --squash                      # Sync beads only, accumulate changes
bd update agent-flywheel-abc.5 --notes "Database schema designed, starting implementation"
bd sync -m "Update progress on database migration"
```

### Handling Beads Conflicts

```shell
# If bd sync encounters conflicts
bd sync                               # May show conflict resolution options

# Check sync status
bd sync --status                      # Show diff between sync branch and main

# Resolve conflicts and complete sync
bd sync -m "Resolve task assignment conflicts"
```

## Epic and Task Management Workflow

### Creating a Comprehensive Epic

```shell
# 1. Sync first to get latest state
bd sync

# 2. Create the epic with description
bd create "API Rate Limiting System" -t epic -p 1 \
  --description="Implement comprehensive rate limiting to prevent API abuse. Include per-user limits, IP-based limits, and Redis-based storage with monitoring dashboards."

# Get the epic ID (e.g., agent-flywheel-def)
epic_id=$(bd list --type=epic --json | jq -r '.[-1].id')

# 3. Create detailed tasks under the epic
bd create "Research rate limiting algorithms" -t task -p 3 --parent="$epic_id" \
  --description="Compare token bucket, sliding window, and fixed window approaches. Document pros/cons and recommend approach."

bd create "Implement Redis rate limiter service" -t task -p 2 --parent="$epic_id" \
  --description="Create core rate limiting service using Redis. Include configurable limits and time windows."

bd create "Add rate limiting middleware" -t task -p 2 --parent="$epic_id" \
  --description="Integrate rate limiter into Express middleware stack. Handle limit exceeded responses."

bd create "Create monitoring dashboard" -t task -p 3 --parent="$epic_id" \
  --description="Build Grafana dashboard showing rate limit metrics, violations, and trends."

bd create "Add rate limit configuration API" -t task -p 3 --parent="$epic_id" \
  --description="Allow admins to update rate limits via API without deployment."

# 4. Set dependencies between tasks
bd update agent-flywheel-def.2 --deps="blocks:agent-flywheel-def.3"  # Redis service blocks middleware
bd update agent-flywheel-def.4 --deps="depends-on:agent-flywheel-def.2"  # Dashboard depends on service

# 5. Sync the complete epic with team
bd sync -m "Add API rate limiting epic with 5 tasks and dependencies"
```

### Epic Progress Tracking

```shell
# Check epic progress
bd show agent-flywheel-def            # Show epic details and all child tasks
bv                                    # Visual progress in kanban

# Update epic description with progress
bd update agent-flywheel-def --notes="Epic 60% complete. Redis service done, middleware in progress."

# Sync progress updates
bd sync -m "Update rate limiting epic progress"
```

## Branch and Sync Strategy

### Using Beads-Sync Branch

```shell
# Check current sync branch configuration
bd config get sync.branch             # Should show "beads-sync"

# Check what's different between main and sync branch
git checkout beads-sync
git log main..beads-sync --oneline     # See beads-only commits

# Merge beads changes back to main (periodically)
git checkout main
bd sync --merge                        # Merge sync branch back to main
```

### Handling Complex Sync Scenarios

```shell
# Force push detected on remote sync branch
bd sync --check                       # Check for issues
bd sync --accept-rebase               # Accept the force push changes

# Working on feature branch without upstream
bd sync --from-main                   # One-way sync from main branch

# Emergency flush before commit (pre-commit hook)
bd sync --flush-only                  # Export to JSONL without git operations
```

## Integration with Git Workflow

### Feature Branch Workflow with Beads

```shell
# 1. Create feature branch for code
git checkout -b feature/user-profiles

# 2. Create beads epic for tracking
bd create "User Profile Management" -t epic -p 2
bd sync -m "Add user profiles epic"

# 3. Work on feature with tasks
bd create "Design user profile schema" -t task -p 3 --parent="agent-flywheel-ghi"
bd update agent-flywheel-ghi.1 --status in_progress

# 4. Develop feature...
# [code development happens]

# 5. Commit code to feature branch
git add .
git commit -m "Add user profile database schema"

# 6. Complete beads task
bd close agent-flywheel-ghi.1
bd sync -m "Complete profile schema design task"

# 7. When feature is complete, merge both code and beads
git checkout main
git merge feature/user-profiles        # Merge code
bd sync --merge                        # Merge any accumulated beads changes
```

### Hotfix Workflow

```shell
# 1. Emergency bug fix
git checkout -b hotfix/api-timeout

# 2. Create high-priority beads issue
bd create "Fix API timeout in user search" -t bug -p 0 \
  --description="Users experiencing 30s timeouts on search. Production issue affecting 15% of requests."

# 3. Update status and sync immediately
bd update agent-flywheel-jkl --status in_progress --assignee="myname"
bd sync -m "Emergency: API timeout hotfix started"

# 4. Quick fix development
ntm spawn hotfix --cc=1               # Single agent for focused work
ntm send hotfix "Fix API timeout in user search endpoint. Check database queries and add indexes if needed."

# 5. Test and deploy
ubs .                                 # Quick bug scan
git add .
git commit -m "Fix API timeout by optimizing user search query"
git push origin hotfix/api-timeout

# 6. Complete and notify team
bd close agent-flywheel-jkl
bd sync -m "Complete emergency API timeout hotfix"
```

## Memory and Context Management

### Building Context for Complex Tasks

```shell
# 1. Start task with context from memory
bd update agent-flywheel-xyz.3 --status in_progress
cm context "authentication JWT tokens React" --json

# 2. Start agents with rich context
ntm spawn jwt-task --cc=2
ntm send jwt-task "Implement JWT token validation middleware. Context: $(cm context 'JWT validation patterns' --json)"

# 3. Update memory as you learn
cm reflect                            # Update procedural memory from recent sessions

# 4. Search for related past work
cass                                  # Search across all session history
```

### Sharing Context with Team

```shell
# Share useful context in beads task notes
bd update agent-flywheel-xyz.3 --notes="Found auth0 JWT library works best. Context: $(cm context 'auth0 JWT' --json | head -c 500)"

bd sync -m "Share JWT implementation context with team"
```

## Monitoring and Quality Assurance

### Pre-Commit Quality Checks

```shell
# Complete quality check workflow
ubs .                                 # Comprehensive bug scan
bd update current-task --notes="Code review passed UBS scan, ready for commit"

# Ensure beads data is synced before code commit
bd sync --flush-only                  # Export any pending beads changes
git add .beads/issues.jsonl           # Include beads in commit
git commit -m "Feature implementation with beads tracking"
```

### Team Quality Reviews

```shell
# Review team's work
bd list --status=needs_review         # See tasks waiting for review
bv                                    # Visual review queue

# Provide feedback in beads
bd update agent-flywheel-xyz.4 --notes="Code looks good, but add unit tests for edge cases"
bd sync -m "Code review feedback for user validation task"
```

## Troubleshooting Workflows

### Beads Sync Issues

```shell
# Check sync status and conflicts
bd sync --status                      # Show differences
bd sync --dry-run                     # Preview what will happen
bd sync --check                       # Detect issues before syncing

# Force resolution
bd sync --accept-rebase               # Accept remote force push
bd sync --import-only                 # Only import, skip git operations
```

### Agent Communication Issues

```shell
# Reset agent environment
ntm list                              # See active sessions
ntm kill session-name                 # Kill problematic session
ntm spawn fresh-session --cc=1 --cod=1 # Start fresh

# Debug agent access
cc "Hello, can you access the project files?" # Test Claude
cod "Show me the current directory" # Test Codex
```

## Best Practices Summary

### Daily Routine
1. **Start day**: `bd sync` (get team updates)
2. **Check work**: `bd ready` and `bv` (see available tasks)
3. **Claim task**: `bd update task-id --status in_progress --assignee myname`
4. **Sync claim**: `bd sync -m "Claim task X"` (notify team immediately)
5. **Work with agents**: Use `ntm` and `cm` for context
6. **End day**: Complete tasks and `bd sync` (share progress)

### Team Coordination
- **Sync frequently**: After claiming tasks, completing work, or major updates
- **Use descriptive commit messages**: Help teammates understand changes
- **Update task notes**: Share context and blockers with the team
- **Review dependencies**: Check `bd show epic-id` for task relationships

### Quality Assurance
- **Run UBS before commits**: Catch issues early
- **Update memory after learning**: Use `cm reflect` to build context
- **Document decisions in beads**: Task notes preserve team knowledge
- **Sync beads data with code**: Keep tracking in sync with implementation

This workflow ensures effective collaboration, maintains project context, and leverages the full power of the agent-flywheel ecosystem.