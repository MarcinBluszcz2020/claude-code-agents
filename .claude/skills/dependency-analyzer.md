---
name: dependency-analyzer
description: Analyzes code dependencies, detects circular dependencies, finds unused imports, and generates dependency graphs. Helps architects and developers understand code structure.
version: 1.0.0
category: quality
priority: high
---

# Dependency Analyzer

Analyzes and visualizes code dependencies for better architecture understanding.

## Usage

### Action: analyze
```
Skill: dependency-analyzer
Action: analyze
Scope: file | module | project
Path: src/auth/
Check Circular: true
Check Unused: true
```

Returns dependency analysis with circular deps, unused imports, and recommendations.

### Action: graph
```
Skill: dependency-analyzer
Action: graph
Path: src/
Output Format: mermaid | json
```

Generates dependency graph visualization.

### Action: circular
```
Skill: dependency-analyzer
Action: circular
Path: src/
```

Specifically checks for circular dependencies.

## Output Example

```
Dependencies Analyzed: 47 files

Circular Dependencies Found: 2
  1. Auth.js → User.js → Auth.js
  2. Service.js → Repository.js → Service.js

Unused Imports: 8
  - lodash in utils.js
  - moment in auth.js

Recommendations:
  - Break circular dependency in Auth/User
  - Remove unused imports to reduce bundle size
```

## Version History
- **1.0.0** (2025-11-16): Initial implementation
