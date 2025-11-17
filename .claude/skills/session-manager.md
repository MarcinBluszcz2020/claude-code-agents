---
name: session-manager
description: Manages session folders with automatic sequential numbering validation. Ensures ZERO errors in session naming by enforcing DDMM_YYYY_NN format without suffixes. Eliminates ~600 lines of session management code.
version: 1.0.0
category: core
priority: critical
---

# Session Manager

A core skill that manages session folder creation, validation, and metadata for all agents. Completely eliminates session naming errors that plague manual management.

## Purpose

**Problem:** Orchestrators have ~300 lines of session management code + manual errors
**Solution:** Automated session management with built-in validation
**Benefit:** ZERO naming errors + eliminates 600+ lines of duplication

## Capabilities

- ✅ List all existing sessions with metadata
- ✅ Find next sequential session number (global across ALL agents)
- ✅ Validate session ID format (DDMM_YYYY_NN, NO SUFFIXES)
- ✅ Create session folders with proper structure
- ✅ Generate session metadata files
- ✅ Detect naming conflicts before creation
- ✅ Track which agent created each session

## Critical Rules

### GLOBAL SEQUENTIAL NUMBERING
- Session numbers are GLOBAL across ALL agents
- NO SUFFIXES allowed (_mvp, _error, _fix, etc.)
- Format: `DDMM_YYYY_NN` where NN is zero-padded (01, 02, ..., 99)
- ALWAYS increment from highest existing number

### Examples
✅ CORRECT:
```
1611_2025_01  (created by mvp-manager)
1611_2025_02  (created by error-manager)
1611_2025_03  (created by mvp-manager)
1611_2025_04  (created by quick-developer)
```

❌ WRONG:
```
1611_2025_01_mvp     (NO SUFFIXES!)
1611_2025_01_error   (NO SUFFIXES!)
1611_2025_error_01   (WRONG POSITION!)
1611_2025_1          (NOT ZERO-PADDED!)
```

## Usage

### Action: list

Lists all existing sessions with metadata.

**Input:**
```
Skill: session-manager
Action: list
```

**Output:**
- sessions: array of session objects
- total_count: number
- latest_session: object

**Example:**
```
Skill: session-manager
Action: list
```

Returns:
```
Sessions Found: 3
Latest: 1611_2025_03

Sessions:
  1. 1611_2025_01 (created by mvp-manager on 2025-11-16 09:30)
  2. 1611_2025_02 (created by error-manager on 2025-11-16 10:15)
  3. 1611_2025_03 (created by mvp-manager on 2025-11-16 11:00)
```

### Action: get-next

Gets the next sequential session ID without creating folder.

**Input:**
```
Skill: session-manager
Action: get-next
Agent Name: <your-agent-name> (optional, for metadata)
```

**Output:**
- next_session_id: string
- previous_sessions: array
- validation: object

**Example:**
```
Skill: session-manager
Action: get-next
Agent Name: error-manager
```

Returns:
```
Next Session ID: 1611_2025_04
Previous Sessions: [1611_2025_01, 1611_2025_02, 1611_2025_03]
Validation: passed
Ready to create: project/internal/1611_2025_04/
```

### Action: validate

Validates a session ID format and checks for conflicts.

**Input:**
```
Skill: session-manager
Action: validate
Session ID: <DDMM_YYYY_NN>
```

**Output:**
- is_valid: boolean
- errors: array
- warnings: array
- exists: boolean

**Example 1 - Valid:**
```
Skill: session-manager
Action: validate
Session ID: 1611_2025_05
```

Returns:
```
Valid: true
Errors: []
Warnings: []
Exists: false
Ready to use: true
```

**Example 2 - Invalid (has suffix):**
```
Skill: session-manager
Action: validate
Session ID: 1611_2025_01_mvp
```

Returns:
```
Valid: false
Errors:
  - Invalid format: contains suffix '_mvp'
  - Expected format: DDMM_YYYY_NN (no suffixes)
Warnings: []
Exists: false
Ready to use: false
```

**Example 3 - Invalid (already exists):**
```
Skill: session-manager
Action: validate
Session ID: 1611_2025_01
```

Returns:
```
Valid: true (format is correct)
Errors: []
Warnings:
  - Session already exists (created by mvp-manager on 2025-11-16 09:30)
  - Use next available number instead
Exists: true
Ready to use: false
```

### Action: create

Creates a new session folder with full structure and metadata.

**Input:**
```
Skill: session-manager
Action: create
Agent Name: <your-agent-name>
Task Description: <what you're doing> (optional, for metadata)
```

**Output:**
- session_id: string
- session_path: string
- metadata_path: string
- created: boolean

**Example:**
```
Skill: session-manager
Action: create
Agent Name: mvp-manager
Task Description: Build MVP authentication system
```

Returns:
```
Session Created: 1611_2025_04
Session Path: project/internal/1611_2025_04/
Metadata File: project/internal/1611_2025_04/000-002-session-metadata.json

Folder Structure Created:
  ✓ project/internal/1611_2025_04/
  ✓ 000-002-session-metadata.json

Metadata:
  - Session ID: 1611_2025_04
  - Created by: mvp-manager
  - Created at: 2025-11-16T12:00:00.000Z
  - Task: Build MVP authentication system
```

### Action: get-metadata

Retrieves metadata for an existing session.

**Input:**
```
Skill: session-manager
Action: get-metadata
Session ID: <DDMM_YYYY_NN>
```

**Output:**
- session_id: string
- created_by: string
- created_at: timestamp
- task_description: string
- status: string
- agents_involved: array

**Example:**
```
Skill: session-manager
Action: get-metadata
Session ID: 1611_2025_01
```

Returns:
```
Session ID: 1611_2025_01
Created By: mvp-manager
Created At: 2025-11-16T09:30:00.000Z
Task: Build MVP authentication system
Status: completed
Agents Involved: [mvp-manager, code-architect, quick-developer]
```

## Session Metadata Format

Every session gets a metadata file: `000-002-session-metadata.json`

```json
{
  "session_id": "1611_2025_04",
  "created_by": "mvp-manager",
  "created_at": "2025-11-16T12:00:00.000Z",
  "task_description": "Build MVP authentication system",
  "current_phase": "000",
  "status": "initializing",
  "agents_involved": ["mvp-manager"],
  "files_created": [],
  "last_updated": "2025-11-16T12:00:00.000Z"
}
```

This file is created automatically by the skill and can be updated as work progresses.

## Validation Rules

### Format Validation
```regex
Pattern: ^\d{4}_\d{4}_\d{2}$
```

Components:
- DDMM: Day (01-31) + Month (01-12)
- YYYY: Year (2025, 2026, etc.)
- NN: Sequential number (01-99), zero-padded

### Forbidden Patterns
- Any suffix after NN: `_mvp`, `_error`, `_fix`, `_test`, etc.
- Non-zero-padded numbers: `1611_2025_1` (should be `01`)
- Wrong separators: `1611-2025-01` (must use underscores)
- Mixed formats: `1611_2025_01_something`

### Validation Checks
1. ✓ Matches regex pattern
2. ✓ No suffixes present
3. ✓ Number is zero-padded
4. ✓ Not already existing
5. ✓ Next in sequence (if creating)

## Workflow Integration

### For Orchestrators (mvp-manager, error-manager)

**Before (without skill) - ~100 lines:**
```markdown
1. Run ls -la project/internal/
2. Manually find highest number
3. Calculate next = highest + 1
4. Verify no suffix
5. Create folder with mkdir
6. Hope you got it right
```

**After (with skill) - ~3 lines:**
```markdown
Skill: session-manager
Action: create
Agent Name: mvp-manager
Task Description: {task}
```

### Mandatory Usage Pattern

**EVERY orchestrator session MUST start with:**
```markdown
## STEP 1: Create Session Folder

Skill: session-manager
Action: create
Agent Name: {your-agent-name}
Task Description: {user-task}

This returns the session_id to use for all subsequent operations.
```

## Error Prevention

### Common Errors Prevented

**Error 1: Suffix Added**
```
❌ Human: Creates 1611_2025_01_mvp
✅ Skill: Validates and rejects, suggests 1611_2025_04
```

**Error 2: Number Reused**
```
❌ Human: Sees 1611_2025_01, 03 and creates 02 (thinking it's free)
✅ Skill: Knows 02 exists, suggests 04
```

**Error 3: Not Zero-Padded**
```
❌ Human: Creates 1611_2025_1
✅ Skill: Auto-formats to 1611_2025_01
```

**Error 4: Wrong Agent Creates Session**
```
❌ Human: error-manager creates folder with _error suffix
✅ Skill: Creates clean sequential number, tracks in metadata who created it
```

## Best Practices

1. **ALWAYS use skill** for session creation (never manual mkdir)
2. **Use create action** at session start
3. **Check metadata** if resuming existing session
4. **List sessions** to understand what exists
5. **Validate** before attempting creation if unsure

## Session Folder Structure

When created, the skill sets up:

```
project/internal/DDMM_YYYY_NN/
└── 000-002-session-metadata.json
```

Additional files added by agents:
```
project/internal/DDMM_YYYY_NN/
├── 000-001-session-report.json       (json-report-manager)
├── 000-002-session-metadata.json     (session-manager)
├── 000-003-task-description.md       (orchestrator)
├── 001-001-architect-task.md         (orchestrator)
├── 001-002-early-architecture.md     (code-architect)
├── 002-001-mapper-task.md            (orchestrator)
├── 002-002-interactions.json         (interaction-mapper)
├── ...
└── 999-001-session-status.md         (orchestrator)
```

## Integration with Other Skills

### Works With:
- **json-report-manager**: Uses session_id for report paths
- **agent-signature-generator**: Includes session_id in signatures
- **file-naming-validator**: Validates files within session folders

### Workflow:
```
1. session-manager: Create session → get session_id
2. json-report-manager: Create report using session_id
3. agent-signature-generator: Sign files with session_id
4. Continue work...
```

## Migration Guide

### For Existing Orchestrators

**Old Pattern:**
```markdown
1. Run `ls -la project/internal/` and LIST all folders you see
2. Identify the HIGHEST number for today's date (DDMM_YYYY_NN)
3. Calculate next number = highest + 1
4. Verify your folder name matches pattern: `^\d{4}_\d{4}_\d{2}$`
5. Confirm NO suffixes in your folder name (no _mvp, _error, etc.)
6. Double-check you're not reusing an existing number
7. mkdir -p project/internal/DDMM_YYYY_NN/
```

**New Pattern:**
```markdown
Skill: session-manager
Action: create
Agent Name: mvp-manager
Task Description: {task}
```

### Benefits of Migration
- **7 manual steps → 1 skill call**
- **100% validation coverage**
- **Zero human error**
- **Automatic metadata generation**
- **Complete audit trail**

## Advanced Usage

### Check What Would Be Next (without creating)
```
Skill: session-manager
Action: get-next
```

Use this when you want to know the next ID but aren't ready to create yet.

### Validate Before Creating
```
Skill: session-manager
Action: validate
Session ID: 1611_2025_05
```

Use this if you have a specific session ID in mind and want to check if it's valid.

### List All Sessions for Audit
```
Skill: session-manager
Action: list
```

Use this to see complete history of all sessions.

## Troubleshooting

### "Session already exists"
```
Problem: Trying to create 1611_2025_03 but it exists
Solution: Use 'get-next' to find next available number
```

### "Invalid format: contains suffix"
```
Problem: Provided session ID like 1611_2025_01_mvp
Solution: Remove suffix, use clean sequential number
```

### "Number not zero-padded"
```
Problem: Provided 1611_2025_1
Solution: Use 1611_2025_01 (zero-padded)
```

### "Can't find project/internal/ directory"
```
Problem: project/internal/ doesn't exist yet
Solution: Skill will create it automatically
```

## Metrics Tracking

The skill tracks:
- Total sessions created today
- Sessions per agent
- Average sessions per day
- Sequential gaps (if any)

Access with:
```
Skill: session-manager
Action: stats
```

## Version History

- **1.0.0** (2025-11-16): Initial implementation with full CRUD + validation
