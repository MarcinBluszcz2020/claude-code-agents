---
name: json-report-manager
description: Comprehensive JSON report management for all agents. Creates, updates, validates and finalizes session reports according to JSON-REPORTING-STANDARD. Eliminates ~1500 lines of duplication across agents.
version: 1.0.0
category: core
priority: critical
---

# JSON Report Manager

A core skill that manages JSON session reports for all agent types (orchestrators, planners, doers). Ensures 100% compliance with JSON-REPORTING-STANDARD.md and eliminates massive code duplication.

## Purpose

**Problem:** Every agent has ~150 lines defining JSON report structure
**Solution:** Single skill that generates, validates, and maintains reports
**Benefit:** Eliminates 1500+ lines of duplication + guarantees consistency

## Capabilities

- ✅ Generate report templates per agent type
- ✅ Validate report structure against standard
- ✅ Auto-fill metadata (timestamp, session_id, agent_name)
- ✅ Update reports (add delegations, errors, files_modified)
- ✅ Calculate metrics (duration, etc.)
- ✅ Finalize reports (set end_time, status)

## Usage

### Action: create

Creates a new JSON report with all required fields.

**Input:**
```
Skill: json-report-manager
Action: create
Agent Type: <orchestrator | planner | doer>
Agent Name: <your-agent-name>
Session ID: <DDMM_YYYY_NN>
Task Description: <what you're doing>
Requested By: <who requested this> (optional, defaults to "user")
```

**Output:**
- JSON report object
- File path where report was saved
- Success/failure message

**Example:**
```
Skill: json-report-manager
Action: create
Agent Type: doer
Agent Name: quick-developer
Session ID: 1611_2025_01
Task Description: Implement MVP authentication system
Requested By: mvp-manager
```

Returns report at: `project/internal/1611_2025_01/004-001-developer-report.json`

### Action: update

Updates an existing report with new data.

**Input:**
```
Skill: json-report-manager
Action: update
Agent Name: <your-agent-name>
Session ID: <DDMM_YYYY_NN>
Report Path: <path-to-existing-report>
Update Data:
  Delegations: [list of delegation objects] (optional)
  Files Modified: [list of file paths] (optional)
  Metrics: {key: value pairs} (optional)
  Errors: [list of error objects] (optional)
  Notes: <string> (optional)
```

**Example:**
```
Skill: json-report-manager
Action: update
Agent Name: mvp-manager
Session ID: 1611_2025_01
Report Path: project/internal/1611_2025_01/000-001-session-report.json
Update Data:
  Delegations:
    - To Agent: code-architect
      Purpose: Design authentication architecture
      Input Files: [001-001-architect-task.md]
      Output Files: [001-002-architecture.md]
      Timestamp: 2025-11-16T10:35:00.000Z
      Status: completed
      Duration Seconds: 300
```

### Action: finalize

Finalizes a report by setting end_time, status, and calculating final metrics.

**Input:**
```
Skill: json-report-manager
Action: finalize
Agent Name: <your-agent-name>
Session ID: <DDMM_YYYY_NN>
Report Path: <path-to-report>
Status: <success | failure | partial> (optional, defaults to "success")
Update Data: <same as update action> (optional)
```

**Example:**
```
Skill: json-report-manager
Action: finalize
Agent Name: quick-developer
Session ID: 1611_2025_01
Report Path: project/internal/1611_2025_01/004-001-developer-report.json
Status: success
Update Data:
  Files Modified:
    - /src/auth/login.js
    - /src/auth/register.js
  Metrics:
    Lines Added: 450
    Lines Removed: 20
    Files Created: 3
  Notes: MVP implementation completed with documented shortcuts
```

### Action: validate

Validates a report against JSON-REPORTING-STANDARD.

**Input:**
```
Skill: json-report-manager
Action: validate
Report Path: <path-to-report>
```

**Output:**
- is_valid: boolean
- errors: array of error messages
- warnings: array of warning messages

**Example:**
```
Skill: json-report-manager
Action: validate
Report Path: project/internal/1611_2025_01/000-001-session-report.json
```

Returns:
```
Valid: false
Errors:
  - Missing required field: task_description
  - Invalid session_id format (has suffix)
Warnings:
  - Orchestrator agent has empty delegations array
```

## Report Templates by Agent Type

### Orchestrator Template
```json
{
  "session_id": "DDMM_YYYY_NN",
  "agent_name": "<agent-name>",
  "requested_by": "user",
  "task_description": "<description>",
  "start_time": "<ISO-8601>",
  "end_time": null,
  "status": "in_progress",
  "delegations": [],
  "files_modified": [],
  "notes": ""
}
```

### Planner Template
```json
{
  "session_id": "DDMM_YYYY_NN",
  "agent_name": "<agent-name>",
  "requested_by": "<orchestrator>",
  "task_description": "<description>",
  "start_time": "<ISO-8601>",
  "end_time": null,
  "status": "in_progress",
  "delegations": [],
  "files_modified": [],
  "metrics": {},
  "notes": ""
}
```

### Doer Template
```json
{
  "session_id": "DDMM_YYYY_NN",
  "agent_name": "<agent-name>",
  "requested_by": "<orchestrator>",
  "task_description": "<description>",
  "start_time": "<ISO-8601>",
  "end_time": null,
  "status": "in_progress",
  "delegations": [],
  "files_modified": [],
  "metrics": {
    "lines_added": 0,
    "lines_removed": 0,
    "files_created": 0
  },
  "notes": ""
}
```

## Validation Rules

### Required Fields (All Agent Types)
- session_id (format: DDMM_YYYY_NN, NO SUFFIXES)
- agent_name
- requested_by
- task_description
- start_time (ISO-8601 format)
- status (in_progress | success | failure | partial)

### Agent-Specific Validation

**Orchestrators (mvp-manager, error-manager):**
- MUST have delegations array
- WARNING if delegations is empty (indicates doing work themselves)
- files_modified should only contain project/internal/ or .claude/ paths

**Planners (code-architect, interaction-mapper, error-tracker):**
- delegations array MUST be empty (they don't delegate)
- files_modified should only contain documentation/specs

**Doers (quick-developer, quality-developer, test-developer):**
- delegations array MUST be empty
- files_modified can contain any paths (they write code)
- MUST have metrics object

### Session ID Validation
- Format: `^\d{4}_\d{4}_\d{2}$`
- NO SUFFIXES like _error, _mvp, _fix
- Examples:
  - ✅ `1611_2025_01`
  - ✅ `1611_2025_02`
  - ❌ `1611_2025_01_mvp`
  - ❌ `1611_2025_error_01`

### Timestamp Validation
- ISO-8601 format with milliseconds: `YYYY-MM-DDTHH:mm:ss.sssZ`
- end_time must be after start_time (if both present)
- Example: `2025-11-16T10:30:00.000Z`

## File Path Conventions

Reports are saved to:
- Orchestrators: `project/internal/{session_id}/000-001-session-report.json`
- Planners: `project/internal/{session_id}/{phase}-001-{agent}-report.json`
  - code-architect: `001-001-architect-report.json` or `003-001-architect-report.json`
  - interaction-mapper: `002-001-mapper-report.json`
  - error-tracker: `001-001-tracker-report.json`
- Doers: `project/internal/{session_id}/00{phase}-001-developer-report.json`
  - quick-developer: `004-001-developer-report.json`
  - quality-developer: `004-001-developer-report.json`
  - test-developer: `005-001-test-report.json`

## Automatic Calculations

When finalizing, the skill automatically calculates:

### Duration
If start_time and end_time are present:
```
duration_seconds = (end_time - start_time).total_seconds()
```
Added to metrics object.

### File Deduplication
When updating files_modified multiple times, the skill:
- Merges all file paths
- Removes duplicates
- Sorts alphabetically
- Uses absolute paths

### Delegation Tracking
Keeps complete audit trail of all delegations in order.

## Error Handling

### Common Errors

**Missing Required Field**
```
Error: Missing required field: task_description
Action: Provide task_description when creating report
```

**Invalid Session ID**
```
Error: Invalid session_id format: 1611_2025_01_mvp (expected DDMM_YYYY_NN)
Action: Remove suffix from session_id
```

**Agent Name Mismatch**
```
Error: Agent name mismatch: expected quick-developer, found mvp-manager
Action: Use correct agent name for update
```

**Timestamp Inconsistency**
```
Error: end_time (10:00) is before start_time (11:00)
Action: Check timestamp values
```

## Integration with Agents

### Before (without skill) - ~150 lines
Agents had to manually define entire JSON structure, validation rules, file paths, etc.

### After (with skill) - ~15 lines

**On Session Start:**
```markdown
Use json-report-manager skill to create your session report:

Skill: json-report-manager
Action: create
Agent Type: doer
Agent Name: quick-developer
Session ID: {from task input}
Task Description: {from task input}
Requested By: {from task input}
```

**During Work:**
```markdown
Update report with progress:

Skill: json-report-manager
Action: update
Agent Name: quick-developer
Session ID: {session_id}
Report Path: {report_path}
Update Data:
  Files Modified: [new files]
  Metrics: {updated metrics}
```

**On Completion:**
```markdown
Finalize report:

Skill: json-report-manager
Action: finalize
Agent Name: quick-developer
Session ID: {session_id}
Report Path: {report_path}
Status: success
Update Data: {final metrics and notes}
```

## Best Practices

1. **Create report IMMEDIATELY** when session starts
2. **Update regularly** as work progresses
3. **Finalize ALWAYS** before completing session
4. **Validate** if you're unsure about format
5. **Use absolute paths** for files_modified
6. **Include context** in notes field

## Metrics Tracking

### Recommended Metrics by Agent Type

**Orchestrators:**
- total_delegations: number
- total_files_created: number (in project/internal/ only)
- session_duration_minutes: number

**Planners:**
- components_designed: number (for architects)
- interactions_mapped: number (for mappers)
- errors_analyzed: number (for trackers)
- trade_offs_documented: number

**Doers:**
- lines_added: number
- lines_removed: number
- files_created: number
- files_modified: number
- shortcuts_taken: number (for quick-developer)
- patterns_implemented: array (for quality-developer)
- tests_created: number (for test-developer)

## Related Skills

- **agent-signature-generator**: Signs all files including reports
- **session-manager**: Creates session folders where reports live
- **file-naming-validator**: Ensures report filenames follow PPP-SSS convention

## Version History

- **1.0.0** (2025-11-16): Initial implementation with full CRUD operations
