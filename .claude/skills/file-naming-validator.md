---
name: file-naming-validator
description: Validates and generates filenames according to PPP-SSS-descriptive-name.ext convention. Ensures consistency across all session files and helps maintain organized project structure.
version: 1.0.0
category: quality
priority: medium
---

# File Naming Validator

Ensures all session files follow the PPP-SSS-descriptive-name.ext naming convention for perfect organization and traceability.

## Usage

### Action: validate
```
Skill: file-naming-validator
Action: validate
File Name: 001-002-early-architecture.md
```

### Action: suggest
```
Skill: file-naming-validator
Action: suggest
Phase: 004
Description: implementation log
Extension: md
```

Returns: `004-002-implementation-log.md`

### Action: list-phase
```
Skill: file-naming-validator
Action: list-phase
Session Path: project/internal/1611_2025_01
Phase: 004
```

Returns all files in phase 004

## Format: PPP-SSS-descriptive-name.ext

- **PPP**: Phase (000-999)
- **SSS**: Step (001-999)
- **descriptive-name**: lowercase-with-hyphens
- **ext**: file extension

## Version History
- **1.0.0** (2025-11-16): Initial implementation
