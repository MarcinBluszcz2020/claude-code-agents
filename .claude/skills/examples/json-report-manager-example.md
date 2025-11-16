# JSON Report Manager - Przykładowa Implementacja Skill

## Metadata

```yaml
skill_name: json-report-manager
version: 1.0.0
category: core
priority: critical
status: example_implementation
author: system-analysis
created: 2025-11-16
```

---

## Opis

Skill do zarządzania raportami JSON we wszystkich agentach. Eliminuje duplikację ~1500 linii kodu i gwarantuje spójność formatów raportów zgodnie z JSON-REPORTING-STANDARD.md.

---

## Interface

### Inputs

```typescript
interface JsonReportManagerInput {
  action: 'create' | 'update' | 'validate' | 'finalize';

  // For create action
  agent_type?: 'orchestrator' | 'planner' | 'doer' | 'optimizer';
  agent_name: string;
  session_id: string;
  task_description: string;
  requested_by?: string;

  // For update action
  report_path?: string;
  update_data?: {
    delegations?: Array<Delegation>;
    errors?: Array<ErrorRecord>;
    files_modified?: Array<string>;
    metrics?: Record<string, any>;
    notes?: string;
  };

  // For validate action
  report_to_validate?: object;

  // Optional overrides
  user_params?: Record<string, any>;
}

interface Delegation {
  to_agent: string;
  purpose: string;
  input_files: string[];
  output_files: string[];
  timestamp: string;
  status: 'pending' | 'in_progress' | 'completed' | 'failed';
  duration_seconds?: number;
}

interface ErrorRecord {
  timestamp: string;
  error: string;
  resolution: string | null;
}
```

### Outputs

```typescript
interface JsonReportManagerOutput {
  success: boolean;

  // For create
  report?: object;
  report_path?: string;

  // For update
  updated_report?: object;

  // For validate
  validation_results?: {
    is_valid: boolean;
    errors: string[];
    warnings: string[];
  };

  // Common
  message: string;
}
```

---

## Przykłady Użycia

### Przykład 1: Tworzenie Raportu dla Doer Agent

**Input:**
```json
{
  "action": "create",
  "agent_type": "doer",
  "agent_name": "quick-developer",
  "session_id": "1611_2025_01",
  "task_description": "Implement MVP authentication system",
  "requested_by": "mvp-manager"
}
```

**Output:**
```json
{
  "success": true,
  "report": {
    "session_id": "1611_2025_01",
    "agent_name": "quick-developer",
    "requested_by": "mvp-manager",
    "task_description": "Implement MVP authentication system",
    "start_time": "2025-11-16T10:30:00.000Z",
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
  },
  "report_path": "project/internal/1611_2025_01/004-001-developer-report.json",
  "message": "Report created successfully for quick-developer"
}
```

---

### Przykład 2: Aktualizacja Raportu z Delegacją (Orchestrator)

**Input:**
```json
{
  "action": "update",
  "agent_name": "mvp-manager",
  "session_id": "1611_2025_01",
  "report_path": "project/internal/1611_2025_01/000-001-session-report.json",
  "update_data": {
    "delegations": [
      {
        "to_agent": "code-architect",
        "purpose": "Design authentication architecture",
        "input_files": ["001-001-architect-task.md"],
        "output_files": ["001-002-architecture.md"],
        "timestamp": "2025-11-16T10:35:00.000Z",
        "status": "completed",
        "duration_seconds": 300
      }
    ]
  }
}
```

**Output:**
```json
{
  "success": true,
  "updated_report": {
    "session_id": "1611_2025_01",
    "agent_name": "mvp-manager",
    "requested_by": "user",
    "task_description": "Build MVP authentication system",
    "start_time": "2025-11-16T10:30:00.000Z",
    "end_time": null,
    "status": "in_progress",
    "delegations": [
      {
        "to_agent": "code-architect",
        "purpose": "Design authentication architecture",
        "input_files": ["001-001-architect-task.md"],
        "output_files": ["001-002-architecture.md"],
        "timestamp": "2025-11-16T10:35:00.000Z",
        "status": "completed",
        "duration_seconds": 300
      }
    ],
    "files_modified": [
      "project/internal/1611_2025_01/000-001-session-report.json",
      "project/internal/1611_2025_01/001-001-architect-task.md"
    ],
    "notes": "Code architect completed architecture design"
  },
  "message": "Report updated successfully with delegation to code-architect"
}
```

---

### Przykład 3: Finalizacja Raportu

**Input:**
```json
{
  "action": "finalize",
  "agent_name": "quick-developer",
  "session_id": "1611_2025_01",
  "report_path": "project/internal/1611_2025_01/004-001-developer-report.json",
  "update_data": {
    "files_modified": [
      "/src/auth/login.js",
      "/src/auth/register.js",
      "/src/auth/middleware.js",
      "/project/internal/1611_2025_01/004-002-implementation-log.md"
    ],
    "metrics": {
      "lines_added": 450,
      "lines_removed": 20,
      "files_created": 3,
      "shortcuts_taken": 8
    },
    "notes": "MVP implementation completed with documented shortcuts"
  }
}
```

**Output:**
```json
{
  "success": true,
  "updated_report": {
    "session_id": "1611_2025_01",
    "agent_name": "quick-developer",
    "requested_by": "mvp-manager",
    "task_description": "Implement MVP authentication system",
    "start_time": "2025-11-16T10:30:00.000Z",
    "end_time": "2025-11-16T11:00:00.000Z",
    "status": "success",
    "delegations": [],
    "files_modified": [
      "/src/auth/login.js",
      "/src/auth/register.js",
      "/src/auth/middleware.js",
      "/project/internal/1611_2025_01/004-002-implementation-log.md"
    ],
    "metrics": {
      "lines_added": 450,
      "lines_removed": 20,
      "files_created": 3,
      "shortcuts_taken": 8,
      "duration_seconds": 1800
    },
    "notes": "MVP implementation completed with documented shortcuts"
  },
  "message": "Report finalized successfully"
}
```

---

### Przykład 4: Walidacja Raportu

**Input:**
```json
{
  "action": "validate",
  "agent_name": "validation-check",
  "report_to_validate": {
    "session_id": "1611_2025_01",
    "agent_name": "mvp-manager",
    "status": "in_progress",
    "delegations": []
  }
}
```

**Output:**
```json
{
  "success": true,
  "validation_results": {
    "is_valid": false,
    "errors": [
      "Missing required field: requested_by",
      "Missing required field: task_description",
      "Missing required field: start_time"
    ],
    "warnings": [
      "Orchestrator agent (mvp-manager) has empty delegations array - this may indicate the agent is doing work directly instead of delegating"
    ]
  },
  "message": "Validation completed with 3 errors and 1 warning"
}
```

---

## Implementacja w Agencie

### PRZED (bez skill) - ~150 linii boilerplate

```markdown
## JSON REPORTING FORMAT (MANDATORY)

Create/update `XXX-001-developer-report.json` in session folder:

```json
{
  "session_id": "DDMM_YYYY_NN",
  "agent_name": "quick-developer",
  "requested_by": "mvp-manager | error-manager",
  "task_description": "Implement [specific feature/fix]",
  "start_time": "2025-08-31T10:00:00.000Z",
  "end_time": "2025-08-31T10:40:00.000Z",
  "status": "success",
  "delegations": [],
  "files_modified": [
    "/src/auth/login.js",
    "/src/auth/register.js"
  ],
  "metrics": {
    "lines_added": 450,
    "lines_removed": 20,
    "files_created": 4
  },
  "notes": "MVP implementation with documented shortcuts"
}
```

Update this report when you complete your work.
```

### PO (ze skill) - ~10 linii + konfiguracja

```markdown
## JSON REPORTING

Use the `json-report-manager` skill to manage your session report:

**On Start:**
```
Skill: json-report-manager
Action: create
Params:
  agent_type: doer
  agent_name: quick-developer
  session_id: [session_id]
  task_description: [task from input]
  requested_by: [who requested this work]
```

**On Completion:**
```
Skill: json-report-manager
Action: finalize
Params:
  agent_name: quick-developer
  session_id: [session_id]
  report_path: [path to report]
  update_data:
    files_modified: [list of modified files]
    metrics: [your metrics]
    notes: [completion notes]
```
```

**Oszczędność:** 140 linii boilerplate + gwarancja spójności!

---

## Logika Wewnętrzna Skill (Pseudo-kod)

```python
class JsonReportManager:
    """
    Core skill for managing JSON reports across all agents.
    """

    TEMPLATES = {
        'orchestrator': {
            'delegations': [],
            'files_modified': [],
            'notes': ''
        },
        'planner': {
            'delegations': [],
            'files_modified': [],
            'metrics': {},
            'notes': ''
        },
        'doer': {
            'delegations': [],
            'files_modified': [],
            'metrics': {
                'lines_added': 0,
                'lines_removed': 0,
                'files_created': 0
            },
            'notes': ''
        }
    }

    def create_report(self, agent_type, agent_name, session_id,
                      task_description, requested_by=None):
        """
        Creates a new JSON report with all required fields.
        """
        # Get base template for agent type
        template = self.TEMPLATES.get(agent_type, self.TEMPLATES['doer'])

        # Build report
        report = {
            'session_id': session_id,
            'agent_name': agent_name,
            'requested_by': requested_by or 'user',
            'task_description': task_description,
            'start_time': self._get_iso_timestamp(),
            'end_time': None,
            'status': 'in_progress',
            **template
        }

        # Validate
        validation = self._validate_report(report)
        if not validation['is_valid']:
            raise ValidationError(validation['errors'])

        # Determine file path
        report_path = self._get_report_path(agent_type, session_id)

        # Write to file
        self._write_json(report_path, report)

        return {
            'success': True,
            'report': report,
            'report_path': report_path,
            'message': f'Report created successfully for {agent_name}'
        }

    def update_report(self, agent_name, session_id, report_path, update_data):
        """
        Updates an existing report with new data.
        """
        # Read existing report
        report = self._read_json(report_path)

        # Validate it's the right agent
        if report.get('agent_name') != agent_name:
            raise ValueError(f"Agent name mismatch: expected {agent_name}, found {report.get('agent_name')}")

        # Apply updates
        if 'delegations' in update_data:
            report['delegations'].extend(update_data['delegations'])

        if 'files_modified' in update_data:
            # Merge and deduplicate
            existing = set(report.get('files_modified', []))
            new_files = set(update_data['files_modified'])
            report['files_modified'] = sorted(existing | new_files)

        if 'metrics' in update_data:
            report.setdefault('metrics', {})
            report['metrics'].update(update_data['metrics'])

        if 'notes' in update_data:
            report['notes'] = update_data['notes']

        if 'errors' in update_data:
            report.setdefault('errors', [])
            report['errors'].extend(update_data['errors'])

        # Write updated report
        self._write_json(report_path, report)

        return {
            'success': True,
            'updated_report': report,
            'message': 'Report updated successfully'
        }

    def finalize_report(self, agent_name, session_id, report_path, update_data):
        """
        Finalizes a report by setting end_time and status.
        """
        # First do regular update
        result = self.update_report(agent_name, session_id, report_path, update_data)

        # Then finalize
        report = result['updated_report']
        report['end_time'] = self._get_iso_timestamp()
        report['status'] = 'success'  # Could be parameterized

        # Calculate duration
        if report.get('metrics'):
            start = self._parse_timestamp(report['start_time'])
            end = self._parse_timestamp(report['end_time'])
            report['metrics']['duration_seconds'] = int((end - start).total_seconds())

        # Write finalized report
        self._write_json(report_path, report)

        return {
            'success': True,
            'updated_report': report,
            'message': 'Report finalized successfully'
        }

    def validate_report(self, report):
        """
        Validates a report against JSON-REPORTING-STANDARD.
        """
        errors = []
        warnings = []

        # Required fields
        required = ['session_id', 'agent_name', 'requested_by',
                   'task_description', 'start_time', 'status']

        for field in required:
            if field not in report:
                errors.append(f'Missing required field: {field}')

        # Session ID format validation
        if 'session_id' in report:
            if not self._validate_session_id(report['session_id']):
                errors.append(f'Invalid session_id format: {report["session_id"]} (expected DDMM_YYYY_NN)')

        # Timestamp validation
        if 'start_time' in report and 'end_time' in report and report['end_time']:
            start = self._parse_timestamp(report['start_time'])
            end = self._parse_timestamp(report['end_time'])
            if end < start:
                errors.append('end_time is before start_time')

        # Agent-specific validation
        agent_name = report.get('agent_name', '')

        # Orchestrators should have delegations
        if any(x in agent_name for x in ['manager', 'fixer', 'orchestrator']):
            if not report.get('delegations'):
                warnings.append(f'Orchestrator agent ({agent_name}) has empty delegations array')

        # Doers should NOT have delegations
        if 'developer' in agent_name:
            if report.get('delegations'):
                warnings.append(f'Doer agent ({agent_name}) has non-empty delegations array')

        return {
            'is_valid': len(errors) == 0,
            'errors': errors,
            'warnings': warnings
        }

    def _get_report_path(self, agent_type, session_id):
        """
        Determines the appropriate file path for the report.
        """
        phase_map = {
            'orchestrator': '000-001-session-report.json',
            'planner': '001-001-planner-report.json',  # or appropriate phase
            'doer': '004-001-developer-report.json'
        }

        filename = phase_map.get(agent_type, '000-001-report.json')
        return f'project/internal/{session_id}/{filename}'

    def _validate_session_id(self, session_id):
        """
        Validates session ID format: DDMM_YYYY_NN
        """
        import re
        pattern = r'^\d{4}_\d{4}_\d{2}$'
        return bool(re.match(pattern, session_id))

    def _get_iso_timestamp(self):
        """
        Returns current timestamp in ISO 8601 format.
        """
        from datetime import datetime
        return datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S.%f')[:-3] + 'Z'

    def _parse_timestamp(self, timestamp_str):
        """
        Parses ISO 8601 timestamp string.
        """
        from datetime import datetime
        return datetime.fromisoformat(timestamp_str.replace('Z', '+00:00'))

    def _read_json(self, path):
        """
        Reads JSON file from path.
        """
        import json
        with open(path, 'r') as f:
            return json.load(f)

    def _write_json(self, path, data):
        """
        Writes JSON data to path with proper formatting.
        """
        import json
        import os

        # Ensure directory exists
        os.makedirs(os.path.dirname(path), exist_ok=True)

        # Write with pretty formatting
        with open(path, 'w') as f:
            json.dump(data, f, indent=2, ensure_ascii=False)
```

---

## Testy (Przykłady)

```python
import pytest
from json_report_manager import JsonReportManager

class TestJsonReportManager:

    def test_create_report_for_doer(self):
        """Test creating a report for doer agent."""
        manager = JsonReportManager()

        result = manager.create_report(
            agent_type='doer',
            agent_name='quick-developer',
            session_id='1611_2025_01',
            task_description='Test task',
            requested_by='mvp-manager'
        )

        assert result['success'] is True
        assert result['report']['agent_name'] == 'quick-developer'
        assert result['report']['status'] == 'in_progress'
        assert result['report']['delegations'] == []
        assert 'metrics' in result['report']

    def test_validate_invalid_session_id(self):
        """Test validation catches invalid session ID format."""
        manager = JsonReportManager()

        report = {
            'session_id': '1611_2025_01_mvp',  # Invalid - has suffix
            'agent_name': 'test',
            'requested_by': 'user',
            'task_description': 'test',
            'start_time': '2025-11-16T10:00:00.000Z',
            'status': 'in_progress'
        }

        validation = manager.validate_report(report)

        assert validation['is_valid'] is False
        assert any('session_id format' in err for err in validation['errors'])

    def test_orchestrator_without_delegations_warning(self):
        """Test warning for orchestrator with no delegations."""
        manager = JsonReportManager()

        report = {
            'session_id': '1611_2025_01',
            'agent_name': 'mvp-manager',
            'requested_by': 'user',
            'task_description': 'test',
            'start_time': '2025-11-16T10:00:00.000Z',
            'status': 'in_progress',
            'delegations': []
        }

        validation = manager.validate_report(report)

        assert len(validation['warnings']) > 0
        assert any('delegations array' in warn for warn in validation['warnings'])
```

---

## Integracja z Istniejącymi Agentami

### Migration Plan

**Faza 1:** Add skill to .claude/skills/
**Faza 2:** Update 1 agent (quick-developer) do używania skill
**Faza 3:** Test i validate
**Faza 4:** Rollout do pozostałych agentów

### Backwards Compatibility

Skill powinien być w stanie:
- Czytać stare raporty (przed skill'em)
- Updatować je do nowego formatu
- Generować nowe raporty w standardowym formacie

---

## Metryki Sukcesu

Po pełnej implementacji skill'a w systemie:

✅ **Code Reduction:** ~1500 linii usuniętych z definicji agentów
✅ **Consistency:** 100% raportów zgodnych z JSON-REPORTING-STANDARD
✅ **Validation:** 0 raportów z błędami formatowania
✅ **Speed:** Tworzenie raportu <100ms
✅ **Maintainability:** Zmiana w standardzie = 1 plik (skill), nie 9 (agenty)

---

**Status:** Example Implementation
**Next Step:** Prototype & Test
**Estimated Effort:** 2-3 days development + 1 week testing
