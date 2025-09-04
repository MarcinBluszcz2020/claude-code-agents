# Claude Code Agent Standards Reference

This folder contains the shared standards and guidelines that are embedded across all Claude Code agents. These documents serve as the source of truth for rules that multiple agents must follow.

## Important Note

**Agents do NOT read these files directly!** The rules documented here must be EMBEDDED directly into each agent's definition file (`.claude/agents/<agent-name>.md`) to be effective. These files serve as:
1. Documentation for humans maintaining the agent system
2. Source material for embedding rules into agents
3. Reference for updates and consistency checks

## Standard Documents

### 1. JSON-REPORTING-STANDARD.md
**Purpose**: Defines the mandatory JSON reporting format for all agents

**Key Rules**:
- Session ID format: `DDMM_YYYY_NN` with NO suffixes
- Required fields for all agent reports
- Agent-specific requirements (orchestrators vs planners vs doers)
- File naming conventions for JSON reports

**Embedded In**:
- ✅ mvp-manager.md (orchestrator)
- ✅ error-fixer.md (orchestrator)
- ✅ code-architect.md (planner)
- ✅ interaction-mapper.md (planner)
- ✅ quick-developer.md (doer)

### 2. ORCHESTRATION-ENFORCEMENT.md
**Purpose**: Enforces separation of concerns and delegation patterns

**Key Rules**:
- Orchestrators CANNOT write application code
- Tool restrictions (no Edit/MultiEdit for orchestrators)
- Write tool limited to `project/internal/` and `.claude/` paths
- Delegation tracking requirements
- Session folder sequential numbering (global across all agents)

**Embedded In**:
- ✅ mvp-manager.md (full path validation and delegation rules)
- ✅ error-fixer.md (full path validation and delegation rules)

### 3. INTER-AGENT-COMMUNICATION-GUIDELINES.md
**Purpose**: Standardizes file formats and communication between agents

**Key Rules**:
- File numbering system: `PPP-SSS-descriptive-name.ext`
- Agent signature requirements for all files
- Format selection (Markdown vs JSON vs YAML)
- Communication patterns between agents

**Embedded In**:
- Partially in all agents (signature requirements)
- Full numbering system in orchestrators

### 4. BUILD-VALIDATION.md
**Purpose**: Enforces build and test validation throughout development workflows

**Key Rules**:
- Mandatory build checks after each iteration
- No task completion with build errors
- Automatic fix iterations for build failures
- Regression testing for error fixes
- Maximum iteration limits before escalation
- JSON reporting of build status

**Embedded In**:
- ✅ mvp-manager.md (build gates and iteration tracking)
- ✅ error-manager.md (regression testing and iteration management)
- ✅ quality-developer.md (build before completion requirement)
- ✅ quick-developer.md (MVP build fixes and validation)

## How to Update Standards

When you need to update a standard that affects multiple agents:

1. **Update the standard document** in this folder
2. **Identify affected agents** using the "Embedded In" lists above
3. **Update each agent's .md file** with the new rules
4. **Add HTML comments** like `<!-- Source: shared-standards/FILENAME.md -->` to track where rules came from
5. **Test the changes** by running the agents with real tasks

## Critical Rules Summary

### Session Folder Naming (All Orchestrators)
```
Format: DDMM_YYYY_NN
- Sequential numbering across ALL agents
- NO suffixes (_error, _mvp, etc.)
- Check ALL existing folders before creating new ones
```

### Path Restrictions (Orchestrators Only)
```
ALLOWED:
- project/internal/**/*
- .claude/**/*

FORBIDDEN:
- src/**/*
- deploy/**/*
- docs/**/*
- Any code files
```

### JSON Report Requirements (All Agents)
```json
{
  "session_id": "DDMM_YYYY_NN",
  "agent_name": "agent-name",
  "requested_by": "who-requested",
  "delegations": [],  // Non-empty for orchestrators
  "files_modified": [] // Absolute paths only
}
```

## Maintenance Checklist

When adding a new agent:
- [ ] Determine agent type (orchestrator/planner/doer)
- [ ] Embed appropriate standards based on type
- [ ] Add HTML source comments for traceability
- [ ] Update this README with the new agent

When updating a standard:
- [ ] Update the standard document
- [ ] Find all agents with embedded rules (check HTML comments)
- [ ] Update each agent's definition
- [ ] Test with real scenarios
- [ ] Update this README if needed

## Build Validation Quick Reference

### Build Commands by Project Type
```bash
# .NET
dotnet build
dotnet test

# Node.js
npm run build
npm test

# Python
python -m py_compile *.py
python -m pytest
```

### Build Status Indicators
- 🟢 Build Success - Clean compilation
- 🟡 Build Warning - Compiled with warnings  
- 🔴 Build Failed - Compilation errors
- ✅ Tests Passing - All tests green
- ⚠️ Tests Failing - Some tests failed
- ❌ Tests Blocked - Cannot run due to build failure

## Future Standards

Potential standards to be added:
- Performance metrics tracking
- Deployment validation rules
- Security scanning requirements
- Code coverage thresholds
- Dependency update protocols

---
*Last Updated: 2025-09-01*
*Maintained by: project-optimizer*