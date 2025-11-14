# Task List: Sistema de Instrucciones de Revisión para GitHub Copilot

Based on: `0001-prd-copilot-reviewer-instructions.md`

## Important Premises

⚠️ **All documentation, templates, and code comments must be written in English.**
- Template files: English
- Documentation files: English
- Code comments: English
- Variable/function names: English
- Copilot instructions can specify response language (e.g., "respond in Spanish") but the instructions themselves must be in English

## Relevant Files

- `.gitignore` - Git ignore file for temporary and generated files
- `.github/copilot-instructions.md` - Main Copilot instructions file with response style, priorities, and format
- `docs/review/security-checklist.md` - Security validation checklist
- `docs/review/testing-checklist.md` - Testing requirements checklist
- `docs/review/performance-checklist.md` - Performance and cost optimization checklist
- `docs/review/reliability-checklist.md` - Reliability and resilience patterns checklist
- `docs/review/readability-checklist.md` - Code readability and DX standards checklist
- `docs/review/code-conventions.md` - Team-specific code conventions (test builders, naming, SBOM)
- `templates/copilot-instructions-template.md` - Customizable base template for teams
- `templates/team-customization-guide.md` - Guide for teams to customize their templates
- `docs/stack-rules/java-kotlin-rules.md` - Specific rules for Kotlin stack
- `docs/stack-rules/react-typescript-rules.md` - Specific rules for React/TypeScript stack
- `docs/metrics/tracking-guide.md` - Guide for tracking review metrics
- `docs/metrics/data-structure.md` - Data structure specification for metrics collection
- `README.md` - Main project documentation and quick start guide

### Notes

- All files must be written in English
- Templates should specify that Copilot can respond in Spanish (es-ES) while instructions remain in English
- Follow Markdown best practices for readability
- Use clear hierarchical structure with headers and lists

## Tasks

- [x] 1.0 Crear estructura base de directorios y archivos de configuración
  - [x] 1.1 Create `.github/` directory if it doesn't exist
  - [x] 1.2 Create `docs/review/` directory structure
  - [x] 1.3 Create `docs/stack-rules/` directory for stack-specific rules
  - [x] 1.4 Create `docs/metrics/` directory for metrics documentation
  - [x] 1.5 Create `templates/` directory for customizable templates
  - [x] 1.6 Create `.gitignore` entries if needed for generated or temporary files

- [x] 2.0 Implementar instrucciones principales de Copilot (.github/copilot-instructions.md)
  - [x] 2.1 Create file header with purpose and scope section
  - [x] 2.2 Define response style section (language: es-ES, professional tone, severity levels)
  - [x] 2.3 Document output format structure (summary, findings with severity, quick checklist)
  - [x] 2.4 Specify review priorities (Security > Reliability > Performance > Testing > Observability > Readability)
  - [x] 2.5 Add cross-cutting rules section with references to specialized checklists
  - [x] 2.6 Define severity levels (Bloqueante, Importante, Sugerencia) with clear criteria
  - [x] 2.7 Add exclusion patterns section (generated files, lockfiles, non-technical markdown)
  - [x] 2.8 Include output template with examples of good feedback
  - [x] 2.9 Add metadata section (owners, last review date, how to propose changes)

- [x] 3.0 Crear checklists especializados por área técnica (security, testing, performance, reliability, readability)
  - [x] 3.1 Create `docs/review/security-checklist.md` with comprehensive security validations
    - [x] 3.1.1 Add secrets and credentials checks (no plaintext, use Key Vault/KMS)
    - [x] 3.1.2 Add input/output validation rules (XSS, SQLi/NoSQLi, SSRF prevention)
    - [x] 3.1.3 Add authentication and authorization checks (token expiration, minimal scopes)
    - [x] 3.1.4 Add dependency vulnerability checks (CVEs, suggest safe versions)
    - [x] 3.1.5 Add encryption requirements (TLS in transit, encryption at rest)
    - [x] 3.1.6 Add PII/secret masking requirements for logs and traces
  - [x] 3.2 Create `docs/review/testing-checklist.md` with testing requirements
    - [x] 3.2.1 Add unit test requirements for new/critical logic
    - [x] 3.2.2 Add contract/API test requirements when schemas change
    - [x] 3.2.3 Add test data requirements (realistic, no real PII)
    - [x] 3.2.4 Add coverage criteria or rationale if not applicable
  - [x] 3.3 Create `docs/review/performance-checklist.md` with optimization rules
    - [x] 3.3.1 Add N+1 query prevention and index requirements
    - [x] 3.3.2 Add pagination/streaming requirements for large responses
    - [x] 3.3.3 Add caching rules with TTL and invalidation strategies
    - [x] 3.3.4 Add complexity requirements (avoid unnecessary O(n²))
  - [x] 3.4 Create `docs/review/reliability-checklist.md` with resilience patterns
    - [x] 3.4.1 Add timeout, retry with backoff, and circuit breaker requirements
    - [x] 3.4.2 Add idempotency requirements for operations with side effects
    - [x] 3.4.3 Add explicit error handling and partial failure management
  - [x] 3.5 Create `docs/review/readability-checklist.md` with code quality standards
    - [x] 3.5.1 Add rules against nested ternaries
    - [x] 3.5.2 Add cyclomatic complexity guidelines
    - [x] 3.5.3 Add naming convention requirements (expressive names)
    - [x] 3.5.4 Add comment guidelines (only when adding context)
    - [x] 3.5.5 Add type/contract visibility requirements

- [x] 4.0 Implementar reglas específicas por stack tecnológico (Kotlin y React/TypeScript)
  - [x] 4.1 Create `docs/stack-rules/java-kotlin-rules.md`
    - [x] 4.1.1 Add null-safety requirements and best practices
    - [x] 4.1.2 Add sealed/data classes usage guidelines
    - [x] 4.1.3 Add repository/DAO rules (parameterized queries, required indexes)
    - [x] 4.1.4 Add dependency injection and configuration best practices
    - [x] 4.1.5 Add logging and exception handling patterns
  - [x] 4.2 Create `docs/stack-rules/react-typescript-rules.md`
    - [x] 4.2.1 Add TypeScript strict mode requirements (avoid `any` unless justified)
    - [x] 4.2.2 Add React hooks rules (proper dependency arrays, custom hooks)
    - [x] 4.2.3 Add component patterns (functional components, prop interfaces)
    - [x] 4.2.4 Add state management best practices (useState, useEffect, context)
    - [x] 4.2.5 Add HTTP client requirements (timeouts, abort controllers, error handling)
  - [x] 4.3 Add extensibility notes for future stack additions (Python, Swift, etc.)

- [x] 5.0 Crear plantillas personalizables y guía de uso para equipos
  - [x] 5.1 Create `templates/copilot-instructions-template.md` as customizable base
    - [x] 5.1.1 Include all sections from main instructions with placeholder comments
    - [x] 5.1.2 Mark clearly customizable sections (team-specific rules, exclusions)
    - [x] 5.1.3 Add inline comments explaining each section's purpose
    - [x] 5.1.4 Include examples of common customizations
  - [x] 5.2 Create `templates/team-customization-guide.md` with step-by-step instructions
    - [x] 5.2.1 Explain how to copy and customize the base template
    - [x] 5.2.2 Document common customization scenarios (different severity levels, additional stacks)
    - [x] 5.2.3 Provide examples of team-specific rules
    - [x] 5.2.4 Add troubleshooting section for common issues
    - [x] 5.2.5 Include best practices for maintaining customized versions
  - [x] 5.3 Update main README.md with quick start guide
    - [x] 5.3.1 Add "Getting Started" section
    - [x] 5.3.2 Add "For Teams" section with customization instructions
    - [x] 5.3.3 Add "Contributing" section for proposing changes
    - [x] 5.3.4 Add examples and use cases

- [x] 6.0 Documentar sistema de métricas y preparar estructura para tracking
  - [x] 6.1 Create `docs/metrics/tracking-guide.md` with metrics overview
    - [x] 6.1.1 Document Phase 1 basic metrics (adoption rate, common issue types)
    - [x] 6.1.2 Document Phase 2 intermediate metrics (review time reduction, false positive rate)
    - [x] 6.1.3 Document Phase 3 advanced metrics (correlation with production bugs, ROI)
    - [x] 6.1.4 Explain how to collect metrics manually initially
  - [x] 6.2 Create `docs/metrics/data-structure.md` with data schema specification
    - [x] 6.2.1 Define required fields in Copilot responses for tracking
    - [x] 6.2.2 Specify data format for metric collection (JSON schema)
    - [x] 6.2.3 Document how to structure severity, area, and resolution data
    - [x] 6.2.4 Add examples of properly formatted metric data
  - [x] 6.3 Add notes on future dashboard integration points
  - [x] 6.4 Document feedback collection process for continuous improvement