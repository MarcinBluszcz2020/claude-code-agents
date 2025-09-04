# JSON Reporting Standard for All Agents

## Overview
This document defines the MANDATORY JSON reporting format that ALL agents must use when creating session reports in `project/internal/`.

## Standard JSON Schema

### Required Fields (All Agents Must Include)

```json
{
  "session_id": "DDMM_YYYY_NN",
  "agent_name": "agent-name-here",
  "requested_by": "user | mvp-manager | error-fixer | other-agent",
  "task_description": "Clear description of what was requested",
  "start_time": "2025-08-31T09:30:00.000Z",
  "end_time": "2025-08-31T10:15:00.000Z",
  "status": "success | failure | partial",
  "delegations": []
}
```

### Optional Fields (Include When Relevant)

```json
{
  "errors": [
    {
      "timestamp": "2025-08-31T09:45:00.000Z",
      "error": "Description of error",
      "resolution": "How it was resolved or null if unresolved"
    }
  ],
  "notes": "Additional context or observations",
  "files_modified": [
    "/absolute/path/to/file1.js",
    "/absolute/path/to/file2.md"
  ],
  "metrics": {
    "lines_added": 150,
    "lines_removed": 30,
    "files_created": 3,
    "tests_passed": 12,
    "tests_failed": 0
  }
}
```

## Field Specifications

### session_id
- Format: `DDMM_YYYY_NN` (NO SUFFIXES ALLOWED)
- Example: `3108_2025_01`, `3108_2025_02`, `3108_2025_03`
- Must match the session folder name
- **CRITICAL**: Sequential numbering across ALL agents
- **FORBIDDEN**: Suffixes like _error, _mvp, _fix, etc.

### agent_name
- Must be the exact agent name from the agent definition
- Examples: `mvp-manager`, `code-architect`, `quick-developer`, `interaction-mapper`, `error-fixer`

### requested_by
- Who initiated this work
- Usually `user` for top-level agents
- For delegated work, use the delegating agent's name

### task_description
- Clear, concise description of the task
- Should be specific enough to understand the work without reading other files

### start_time / end_time
- ISO 8601 format with millisecond precision
- UTC timezone (Z suffix)
- Format: `YYYY-MM-DDTHH:mm:ss.sssZ`
- end_time is null while work is in progress

### status
- `success`: Task completed successfully
- `failure`: Task could not be completed
- `partial`: Some objectives met but not all
- `in_progress`: Work ongoing (only valid when end_time is null)

### delegations (For Orchestrator Agents)

Required for orchestrator agents (mvp-manager, error-fixer):

```json
"delegations": [
  {
    "to_agent": "code-architect",
    "purpose": "Analyze system architecture",
    "input_files": ["001-001-task.md"],
    "output_files": ["001-002-analysis.md"],
    "timestamp": "2025-08-31T09:31:00.000Z",
    "status": "completed",
    "duration_seconds": 180
  }
]
```

For doer agents, this array should be empty: `"delegations": []`

### errors
- Array of error objects encountered during execution
- Include timestamp for each error
- Document resolution if the error was resolved

### files_modified
- Complete list of all files created or modified
- Use absolute paths only
- Include ALL files, not just source code

### metrics
- Optional performance/quality metrics
- Customize based on agent type and task
- Examples: lines of code, test results, performance benchmarks

## Agent-Specific Requirements

### Orchestrator Agents (mvp-manager, error-fixer)
- MUST have non-empty `delegations` array
- MUST NOT have implementation files in `files_modified`
- MUST track all agent handoffs in delegations

### Planner Agents (code-architect, interaction-mapper)
- `files_modified` should only contain documentation/specification files
- Should include analysis metrics when relevant
- `delegations` should be empty (they don't delegate)

### Doer Agents (quick-developer)
- MUST list all code files in `files_modified`
- Should include implementation metrics
- Should document who requested the work in `requested_by`

## File Naming Convention

JSON reports should be named:
- Primary report: `000-001-session-report.json`
- Updates: `000-002-session-report-update.json`
- Final: `999-001-session-report-final.json`

## Validation Rules

1. **Timestamp Consistency**: start_time must be before end_time
2. **Status Consistency**: If end_time is null, status must be "in_progress"
3. **Delegation Tracking**: Orchestrators must have delegations
4. **File Attribution**: files_modified must be absolute paths
5. **Agent Identification**: agent_name must match known agents

## Example Reports

### Orchestrator Agent (mvp-manager)
```json
{
  "session_id": "3108_2025_01",
  "agent_name": "mvp-manager",
  "requested_by": "user",
  "task_description": "Build MVP for user authentication system",
  "start_time": "2025-08-31T09:30:00.000Z",
  "end_time": "2025-08-31T11:45:00.000Z",
  "status": "success",
  "delegations": [
    {
      "to_agent": "code-architect",
      "purpose": "Design authentication architecture",
      "input_files": ["001-001-architect-task.md"],
      "output_files": ["001-002-architecture.md"],
      "timestamp": "2025-08-31T09:35:00.000Z",
      "status": "completed",
      "duration_seconds": 300
    },
    {
      "to_agent": "quick-developer",
      "purpose": "Implement authentication system",
      "input_files": ["004-001-implementation-task.md"],
      "output_files": ["src/auth/login.js", "src/auth/register.js"],
      "timestamp": "2025-08-31T10:00:00.000Z",
      "status": "completed",
      "duration_seconds": 2400
    }
  ],
  "files_modified": [
    "/project/internal/3108_2025_01/000-001-session-report.json",
    "/project/internal/3108_2025_01/000-002-task-description.md"
  ],
  "notes": "MVP authentication completed with basic email/password support"
}
```

### Doer Agent (quick-developer)
```json
{
  "session_id": "3108_2025_01",
  "agent_name": "quick-developer",
  "requested_by": "mvp-manager",
  "task_description": "Implement user authentication based on architecture spec",
  "start_time": "2025-08-31T10:00:00.000Z",
  "end_time": "2025-08-31T10:40:00.000Z",
  "status": "success",
  "delegations": [],
  "files_modified": [
    "/src/auth/login.js",
    "/src/auth/register.js",
    "/src/auth/middleware.js",
    "/src/models/User.js",
    "/project/internal/3108_2025_01/004-002-implementation-log.md"
  ],
  "metrics": {
    "lines_added": 450,
    "files_created": 4,
    "functions_created": 12
  },
  "notes": "Implemented MVP authentication with JWT tokens and bcrypt hashing"
}
```

## Implementation Checklist

For agent maintainers updating their agents:

- [ ] Add JSON report creation at session start
- [ ] Include all required fields
- [ ] Update report after each major action
- [ ] Track delegations (orchestrators only)
- [ ] List all modified files with absolute paths
- [ ] Set proper status on completion
- [ ] Include relevant metrics
- [ ] Sign all work with agent name

## Enforcement

Agents that fail to follow this standard:
1. Will be flagged in system audits
2. May have their work rejected by orchestrators
3. Will be updated to enforce compliance

This standard ensures complete traceability, accountability, and quality control across the entire agent ecosystem.