# BUILD VALIDATION STANDARD

## Overview
This standard defines mandatory build and test validation requirements for all development workflows. Build validation ensures code quality and prevents accumulation of technical debt.

## Core Principles

1. **No Broken Builds**: Code cannot be marked complete with build errors
2. **Continuous Validation**: Build checks after every iteration
3. **Regression Prevention**: Tests verify fixes don't break existing functionality
4. **Automatic Error Detection**: Build failures trigger immediate fix requirements

## Build Validation Requirements

### When to Run Build Validation

#### MANDATORY Build Checks:
1. **After Each Development Iteration**
   - Run `dotnet build` immediately after implementation
   - Check for compilation errors
   - Verify all references resolve

2. **Before Marking Complete**
   - No task can be marked "completed" with build errors
   - Build must succeed with zero errors
   - Warnings should be documented

3. **After Error Fixes**
   - Verify fix resolves the issue
   - Ensure no new errors introduced
   - Run regression tests if available

### Build Commands by Project Type

#### .NET Projects
```bash
# Basic build
dotnet build

# Build with specific configuration
dotnet build --configuration Release

# Clean and build
dotnet clean && dotnet build

# Build specific project
dotnet build src/ProjectName.csproj
```

#### Node.js Projects
```bash
# TypeScript compilation
npm run build
# or
tsc

# Webpack build
npm run webpack

# Generic build script
npm run build
```

#### Python Projects
```bash
# Syntax check
python -m py_compile *.py

# Type checking with mypy
mypy .

# Linting
pylint *.py
```

## Test Execution Requirements

### When to Run Tests

1. **After Build Success**
   - Only run tests if build succeeds
   - Document test results in JSON report

2. **For Error Fixes**
   - Run relevant unit tests
   - Execute integration tests if affected
   - Verify fix doesn't break existing tests

### Test Commands

#### .NET Projects
```bash
# Run all tests
dotnet test

# Run with verbosity
dotnet test --logger "console;verbosity=detailed"

# Run specific test project
dotnet test tests/ProjectName.Tests.csproj
```

#### Node.js Projects
```bash
# Run tests
npm test

# Run with coverage
npm run test:coverage

# Run specific test suite
npm test -- --grep "AuthenticationTests"
```

## JSON Reporting Format

### Build Validation Report Structure
```json
{
  "build_validation": {
    "timestamp": "2025-09-01T10:00:00.000Z",
    "iteration": 1,
    "build_status": "success | failure",
    "build_command": "dotnet build",
    "build_errors": [],
    "build_warnings": [
      {
        "file": "src/Auth.cs",
        "line": 45,
        "message": "Variable 'temp' is never used",
        "code": "CS0219"
      }
    ],
    "test_status": "success | failure | skipped",
    "test_command": "dotnet test",
    "tests_run": 42,
    "tests_passed": 40,
    "tests_failed": 2,
    "test_failures": [
      {
        "test": "AuthenticationTests.LoginWithInvalidCredentials",
        "error": "Expected false but got true"
      }
    ],
    "action_taken": "Fixed compilation errors in Auth.cs",
    "next_iteration_required": true,
    "fix_description": "Need to resolve test failures in authentication module"
  }
}
```

## Workflow Integration

### For mvp-manager Workflow

```markdown
## Iteration Workflow with Build Validation

1. **Phase 4: Implementation**
   - Developer implements features
   - **BUILD CHECK**: Run `dotnet build`
   - If errors: Document in JSON, proceed to fix iteration
   - If success: Continue to tests

2. **Phase 4.5: Build Validation Gate**
   - **MANDATORY**: Cannot proceed without successful build
   - Document build status in session report
   - If failed: Create fix task for next iteration

3. **Phase 5: Test Execution**
   - Run `dotnet test` only if build succeeds
   - Document test results
   - Failed tests don't block MVP but must be documented

4. **Phase 6: Fix Iteration (if needed)**
   - Fix build errors first priority
   - Fix test failures second priority
   - Re-run build validation
   - Repeat until clean build
```

### For error-manager Workflow

```markdown
## Error Fix Workflow with Validation

1. **Phase 3: Implementation**
   - Developer implements fix
   - **BUILD CHECK**: Immediate validation required

2. **Phase 3.5: Build Validation Gate**
   - **CRITICAL**: Fix cannot be accepted with build errors
   - Must verify fix doesn't introduce new errors
   - Run regression tests

3. **Phase 4: Regression Testing**
   - Verify existing functionality still works
   - Run full test suite if available
   - Document any new test failures

4. **Phase 5: Iteration (if needed)**
   - Fix any build errors introduced
   - Resolve regression test failures
   - Continue until stable
```

## Error Handling Protocol

### Build Failure Response

When build fails, agents MUST:

1. **Capture Error Details**
```json
{
  "build_error": {
    "file": "src/Database/Connection.cs",
    "line": 234,
    "column": 15,
    "error_code": "CS0246",
    "message": "The type or namespace name 'IDbConnection' could not be found"
  }
}
```

2. **Create Fix Task**
```markdown
## Build Error Fix Required

**Error**: CS0246 - Missing type 'IDbConnection'
**Location**: src/Database/Connection.cs:234
**Required Action**: Add using statement or install package
**Priority**: BLOCKING - Must fix before continuing
```

3. **Track in Iteration Report**
```json
{
  "iteration": 2,
  "purpose": "Fix build errors from iteration 1",
  "errors_to_fix": ["CS0246", "CS1061"],
  "estimated_time": "10 minutes"
}
```

## Validation Gates

### Gate 1: Post-Implementation
- **Requirement**: Build must succeed
- **Blocker**: Yes - cannot proceed with build errors
- **Action**: Fix immediately in next iteration

### Gate 2: Pre-Completion
- **Requirement**: Clean build, no errors
- **Blocker**: Yes - cannot mark complete
- **Action**: Document remaining warnings

### Gate 3: Post-Fix
- **Requirement**: Fix resolves issue without breaking build
- **Blocker**: Yes - fix rejected if introduces errors
- **Action**: Iterate until clean

## Implementation Checklist

For Orchestrator Agents (mvp-manager, error-manager):
- [ ] Add build validation step after each developer iteration
- [ ] Create JSON structure for build results
- [ ] Implement iteration loop for build failures
- [ ] Add validation gates to workflow
- [ ] Document build status in reports

For Developer Agents (quick-developer, quality-developer):
- [ ] Run build before marking complete
- [ ] Capture and report build errors
- [ ] Fix compilation errors as top priority
- [ ] Document workarounds for warnings
- [ ] Include build status in completion report

## Build Status Indicators

Use these in status reports:
- 🟢 **Build Success**: Clean compilation, no errors
- 🟡 **Build Warning**: Compiled with warnings
- 🔴 **Build Failed**: Compilation errors present
- 🔄 **Build In Progress**: Currently compiling
- ✅ **Tests Passing**: All tests green
- ⚠️ **Tests Failing**: Some tests failed
- ❌ **Tests Blocked**: Cannot run due to build failure

## Sample Integration

### In mvp-manager Session Report
```json
{
  "iterations": [
    {
      "number": 1,
      "developer": "quick-developer",
      "build_status": "failed",
      "errors": 3,
      "action": "Creating iteration 2 for fixes"
    },
    {
      "number": 2,
      "developer": "quick-developer",
      "build_status": "success",
      "warnings": 2,
      "tests_run": true,
      "tests_passed": 18,
      "tests_failed": 2
    }
  ]
}
```

## Enforcement

This standard is MANDATORY for:
- All mvp-manager workflows
- All error-manager workflows
- Both quick-developer and quality-developer implementations

Non-compliance indicators:
- Marking work complete without build validation
- Proceeding past build errors without fixes
- No build status in JSON reports
- Missing iteration for build failures

Remember: **A broken build is a broken project.** No exceptions.