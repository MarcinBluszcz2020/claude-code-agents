---
name: user-parameters-manager
description: Manages user parameters across all agents. Reads, validates, merges global and agent-specific params, and provides defaults. Centralized configuration management.
version: 1.0.0
category: core
priority: medium
---

# User Parameters Manager

Centralizes user parameters management for consistent configuration across all agents.

## Usage

### Action: read
```
Skill: user-parameters-manager
Action: read
Session Path: project/internal/1611_2025_01
Agent Name: quick-developer
```

Returns merged global + agent-specific parameters with defaults.

### Action: validate
```
Skill: user-parameters-manager
Action: validate
Session Path: project/internal/1611_2025_01
```

Validates user-params.json structure.

### Action: create-template
```
Skill: user-parameters-manager
Action: create-template
Agent Name: mvp-manager
```

Generates template user-params.json.

## Parameters Structure

```json
{
  "global_params": {
    "quality_level": "mvp | production | enterprise",
    "time_constraint": "urgent | normal | flexible"
  },
  "agent_params": {
    "quick-developer": {
      "shortcuts_allowed": true,
      "quality_level": "mvp"
    }
  }
}
```

## Version History
- **1.0.0** (2025-11-16): Initial implementation
