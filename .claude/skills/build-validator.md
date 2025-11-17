---
name: build-validator
description: Universal build and test validation for all project types. Auto-detects project type (.NET, Node.js, Python), runs appropriate build commands, parses results, and generates structured reports. Eliminates ~1200 lines of duplicate build validation code.
version: 1.0.0
category: quality
priority: high
---

# Build Validator

A comprehensive skill that validates builds and tests across all project types. Used by developers and orchestrators to ensure code quality.

## Purpose

**Problem:** 4 agents have ~400 lines each of build validation code
**Solution:** Universal build validator with auto-detection
**Benefit:** Eliminates 1200+ lines + consistent validation

## Capabilities

- ✅ Auto-detect project type (.NET, Node.js, Python, Java, Go, Rust)
- ✅ Run appropriate build commands
- ✅ Parse build output (errors, warnings)
- ✅ Run tests after successful build
- ✅ Parse test results
- ✅ Generate structured JSON reports
- ✅ Track build iterations
- ✅ Suggest fixes for common errors

## Usage

### Action: validate

Runs full build and test validation.

**Input:**
```
Skill: build-validator
Action: validate
Project Path: <path> (optional, defaults to ".")
Run Tests: <boolean> (optional, defaults to true)
Iteration: <number> (optional, for tracking fixes)
```

**Output:**
- build_status: "success" | "failure"
- build_command: string
- build_errors: array
- build_warnings: array
- test_status: "success" | "failure" | "skipped"
- test_results: object
- recommendations: array

**Example:**
```
Skill: build-validator
Action: validate
Project Path: .
Run Tests: true
Iteration: 1
```

Returns:
```
Project Type: dotnet
Build Command: dotnet build
Build Status: success
Build Errors: 0
Build Warnings: 3

Warnings:
  - CS0219: Variable 'temp' is never used (Auth.cs:45)
  - CS0168: Variable 'ex' is declared but never used (UserService.cs:67)
  - CS1998: Async method lacks 'await' (DataService.cs:123)

Test Command: dotnet test
Test Status: success
Tests Run: 42
Tests Passed: 40
Tests Failed: 2

Failed Tests:
  - AuthTests.LoginWithInvalidCredentials
  - UserTests.CreateDuplicateUser

Recommendations:
  - Fix unused variables to clean warnings
  - Review failed tests in authentication module
```

### Action: detect-project-type

Auto-detects project type without running build.

**Input:**
```
Skill: build-validator
Action: detect-project-type
Project Path: <path> (optional, defaults to ".")
```

**Output:**
- project_type: string
- confidence: number (0-1)
- indicators: array

**Example:**
```
Skill: build-validator
Action: detect-project-type
Project Path: .
```

Returns:
```
Project Type: nodejs
Confidence: 0.95

Indicators Found:
  ✓ package.json exists
  ✓ node_modules/ directory exists
  ✓ npm-lock.json or yarn.lock found
  ✓ tsconfig.json (TypeScript project)
```

### Action: build-only

Runs only the build without tests.

**Input:**
```
Skill: build-validator
Action: build-only
Project Path: <path> (optional)
Configuration: <debug | release> (optional, for .NET)
```

**Output:**
- build_status: "success" | "failure"
- build_output: string
- errors: array
- warnings: array

### Action: test-only

Runs only tests (assumes build already succeeded).

**Input:**
```
Skill: build-validator
Action: test-only
Project Path: <path> (optional)
Test Filter: <pattern> (optional, runs specific tests)
```

**Output:**
- test_status: "success" | "failure"
- tests_run: number
- tests_passed: number
- tests_failed: number
- failures: array

## Supported Project Types

### .NET Projects

**Detection:**
- *.csproj, *.sln files
- *.cs source files

**Build Command:**
```bash
dotnet build
```

**Test Command:**
```bash
dotnet test
```

**Error Patterns:**
```
CS0246: Type not found
CS1061: Method does not exist
CS0029: Cannot convert type
```

### Node.js Projects

**Detection:**
- package.json
- node_modules/
- *.js, *.ts files

**Build Command:**
```bash
# TypeScript
tsc
# or
npm run build

# Webpack
npm run webpack
```

**Test Command:**
```bash
npm test
# or
npm run test:coverage
```

**Error Patterns:**
```
TS2304: Cannot find name
TS2345: Argument type mismatch
Module not found
```

### Python Projects

**Detection:**
- requirements.txt or Pipfile
- setup.py
- *.py files

**Build Command:**
```bash
python -m py_compile *.py
# or
python setup.py build
```

**Test Command:**
```bash
pytest
# or
python -m unittest
```

**Error Patterns:**
```
SyntaxError
IndentationError
ModuleNotFoundError
```

### Java Projects

**Detection:**
- pom.xml (Maven) or build.gradle (Gradle)
- *.java files

**Build Command:**
```bash
# Maven
mvn compile

# Gradle
gradle build
```

**Test Command:**
```bash
# Maven
mvn test

# Gradle
gradle test
```

### Go Projects

**Detection:**
- go.mod
- *.go files

**Build Command:**
```bash
go build ./...
```

**Test Command:**
```bash
go test ./...
```

### Rust Projects

**Detection:**
- Cargo.toml
- *.rs files

**Build Command:**
```bash
cargo build
```

**Test Command:**
```bash
cargo test
```

## Build Report Format

```json
{
  "build_validation": {
    "timestamp": "2025-11-16T12:00:00.000Z",
    "iteration": 1,
    "project_type": "dotnet",
    "build_status": "success",
    "build_command": "dotnet build",
    "build_duration_ms": 3450,
    "build_errors": [],
    "build_warnings": [
      {
        "code": "CS0219",
        "file": "src/Auth.cs",
        "line": 45,
        "message": "Variable 'temp' is never used"
      }
    ],
    "test_status": "success",
    "test_command": "dotnet test",
    "test_duration_ms": 1250,
    "tests_run": 42,
    "tests_passed": 42,
    "tests_failed": 0,
    "test_failures": [],
    "coverage_percentage": 85.5,
    "ready_for_deployment": true
  }
}
```

## Iteration Tracking

For tracking build fixes across iterations:

```json
{
  "iterations": [
    {
      "number": 1,
      "build_status": "failure",
      "errors_count": 3,
      "action": "Creating fix task"
    },
    {
      "number": 2,
      "build_status": "failure",
      "errors_count": 1,
      "action": "One error remaining"
    },
    {
      "number": 3,
      "build_status": "success",
      "warnings_count": 2,
      "action": "Build succeeded, proceeding"
    }
  ]
}
```

## Error Parsing

### .NET Errors

```
src/Auth.cs(45,15): error CS0246: The type or namespace name 'IAuthService' could not be found
```

Parsed to:
```json
{
  "code": "CS0246",
  "file": "src/Auth.cs",
  "line": 45,
  "column": 15,
  "message": "The type or namespace name 'IAuthService' could not be found",
  "severity": "error"
}
```

### TypeScript Errors

```
src/auth.ts:45:15 - error TS2304: Cannot find name 'AuthService'.
```

Parsed to:
```json
{
  "code": "TS2304",
  "file": "src/auth.ts",
  "line": 45,
  "column": 15,
  "message": "Cannot find name 'AuthService'",
  "severity": "error"
}
```

## Recommendations Engine

Based on errors, the skill suggests fixes:

### Missing Type/Import
```
Error: CS0246 - Type 'IAuthService' not found
Recommendation: Add using statement or install package
Suggested Fix: using MyApp.Interfaces;
```

### Null Reference
```
Error: NullReferenceException
Recommendation: Add null checks
Suggested Fix: if (user != null) { ... }
```

### Type Mismatch
```
Error: Cannot convert string to int
Recommendation: Parse or validate input type
Suggested Fix: int.Parse(value) or int.TryParse(value, out int result)
```

## Best Practices

1. **Run after every implementation** - catch errors early
2. **Fix build before tests** - tests can't run if build fails
3. **Track iterations** - know how many tries it took
4. **Document warnings** - don't ignore them
5. **Run full suite** - not just unit tests

## Integration Patterns

### With mvp-manager

```
# After developer completes implementation
Skill: build-validator
Action: validate
Project Path: .
Run Tests: false  # MVP may skip tests
Iteration: 1

# If build fails
Create fix task for developer
Increment iteration
Repeat until success
```

### With error-manager

```
# After fix implementation
Skill: build-validator
Action: validate
Project Path: .
Run Tests: true  # MUST run for error fixes
Iteration: 1

# Check for regressions
If new test failures → fix again
If build fails → fix properly
Repeat until clean
```

### With Developers

```
# Before marking work complete
Skill: build-validator
Action: validate
Project Path: .
Run Tests: true
Iteration: {current_iteration}

# Cannot complete with build errors
If build fails → fix immediately
If tests fail → document or fix
```

## Version History

- **1.0.0** (2025-11-16): Initial implementation with multi-language support
