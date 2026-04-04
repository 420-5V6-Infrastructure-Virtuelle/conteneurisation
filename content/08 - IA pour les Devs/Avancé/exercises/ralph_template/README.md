---
title: "Structure du Template"
weight: 1
---

# Ralph Loop Template Structure

This template provides a starting structure for autonomous agent loops with persistent state tracking.

## Directory Structure

```
.project/
├── prd.md              # Product Requirements Document
├── constraints.md      # Technical and business constraints
├── state.json         # Current state (modified by agent)
├── decisions.md       # Decision log (modified by agent)
├── attempts/         # Failed attempts archive
│   └── YYYY-MM-DD_HH-MM/
│       ├── diff.patch
│       └── notes.md
└── completed.json     # Success markers
```

## Files Explained

### `prd.md` - Requirements Specification

```markdown
# Feature: User Notification System

## Summary
Users should receive email notifications when their order status changes.

## Acceptance Criteria
- [ ] Email sent when order status changes to "shipped"
- [ ] Email sent when order status changes to "delivered"
- [ ] Email includes order details and tracking link
- [ ] Users can opt out of notifications in settings
- [ ] Failed emails are retried 3 times with exponential backoff

## Out of Scope
- SMS notifications
- Push notifications
- Marketing emails

## Technical Notes
- Use existing email service (Resend)
- Store notification preferences in user_settings table
- Add notification_log table for audit
```

### `constraints.md` - Technical Constraints

```markdown
# Technical Constraints

## Stack
- Backend: FastAPI
- Database: PostgreSQL + SQLAlchemy
- Email: Resend API
- Background tasks: Celery + Redis

## Must Follow
- Use existing user_settings model
- Follow logging standards (app.logging.get_logger)
- All DB changes via migrations (alembic)
- Tests for all new functions

## Must NOT
- Create new database connections
- Use deprecated send_email() function
- Modify order model directly
- Add new Celery queue (use existing 'notifications' queue)

## Performance
- Max 100 emails per minute (Resend limit)
- Track retry attempts to avoid infinite loops
```

### `state.json` - Current State (Agent Modifies This)

```json
{
  "status": "in_progress",
  "current_task": "Add notification preference fields to user_settings",
  "phase": "implementation",
  "started_at": "2024-01-15T10:30:00Z",
  "last_updated": "2024-01-15T11:45:00Z",
  "context": {
    "files_modified": ["app/models/user_settings.py", "app/schemas/user.py"],
    "files_created": [],
    "tests_passing": true,
    "lint_clean": false
  },
  "next_steps": [
    "Fix lint errors in app/models/user_settings.py",
    "Create notification service",
    "Add tests for notification service"
  ]
}
```

### `decisions.md` - Decision Log

```markdown
# Decision Log

## 2024-01-15 10:32 - Started Implementation
**Agent:** Claude
**Decision:** Start with user_settings model modification
**Rationale:** Foundation for notification preferences

## 2024-01-15 10:45 - Model Updated
**Decision:** Add `notification_preferences` JSON column instead of separate table
**Rationale:** 
- Fewer migrations
- Flexible schema for future notification types
- Consistent with existing user_settings pattern
**Files Changed:** app/models/user_settings.py

## 2024-01-15 11:20 - Service Architecture
**Decision:** Create `NotificationService` class vs direct Celery task calls
**Rationale:**
- Easier testing
- Better separation of concerns
- Consistent with existing service pattern (OrderService, UserService)
```

### `attempts/` - Failed Attempts Archive

```
attempts/
└── 2024-01-15_09-30/
    ├── diff.patch        # What was tried
    └── notes.md          # Why it failed
```

```markdown
# Attempt: 2024-01-15 09:30

## Goal
Send email directly from order status update handler

## Approach
- Import send_notification in app/services/order.py
- Call directly in update_order_status()

## Why It Failed
- Circular dependency: order.py imports notification.py imports order.py
- Tests failed with ImportError
- Lint errors due to import cycle

## Lessons Learned
- Create separate notification_service.py instead
- Use Celery task to avoid direct dependency
- Need to restructure imports
```

### `completed.json` - Success Markers

```json
{
  "completed_at": "2024-01-15T14:30:00Z",
  "summary": "Successfully implemented notification system",
  "files_changed": {
    "modified": [
      "app/models/user_settings.py",
      "app/schemas/user.py",
      "app/services/order.py"
    ],
    "created": [
      "app/services/notification.py",
      "app/tasks/notifications.py",
      "tests/test_notification.py"
    ]
  },
  "tests": {
    "unit": "12 tests, all passing",
    "integration": "3 tests, all passing"
  },
  "manual_verification": "Emails sent successfully in staging environment"
}
```

## Usage

### Starting a Ralph Loop

1. Copy this template directory to your project:
   ```bash
   cp -r exercises/ralph_template/.project your-project/
   ```

2. Edit `prd.md` with your feature requirements
3. Edit `constraints.md` with your project's technical constraints
4. Initialize `state.json`:
   ```json
   {
     "status": "ready",
     "current_task": null,
     "phase": "planning",
     "started_at": null
   }
   ```

5. Start the agent:
   ```
   Ask the agent to read .project/prd.md and .project/constraints.md, 
   then implement the feature step by step, updating state.json as 
   it progresses.
   ```

### Monitoring Progress

```bash
# Check current status
cat .project/state.json | jq '.status, .current_task, .phase'

# View decisions log
tail -f .project/decisions.md

# Check for failed attempts
ls -la .project/attempts/
```

### Recovery After Failure

1. Agent creates a new attempt folder with:
   - What was tried (diff.patch)
   - Why it failed (notes.md)

2. Agent does `git checkout .` to clean state

3. Agent reads previous attempts to avoid repeating mistakes

4. Agent updates approach and tries again

### Success Criteria

The loop should automatically stop when:
- All acceptance criteria in `prd.md` are checked
- `state.json` has status: "completed"
- All tests pass
- Lint clean
- `completed.json` is populated

## Exercise: Module 7

Create a `.project/` directory for a simple feature:

1. Write `prd.md` for: "Add user avatar upload feature"
2. Write `constraints.md` for your demo app
3. Initialize empty `state.json`
4. Ask agent to implement while updating state
5. Observe: Does agent follow the structure? Where does it deviate?