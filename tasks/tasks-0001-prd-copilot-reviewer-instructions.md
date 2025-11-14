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
- `templates/copilot-instructions-template.md` - Customizable base template for teams
- `templates/team-customization-guide.md` - Guide for teams to customize their templates
- `docs/stack-rules/java-kotlin-rules.md` - Specific rules for Java/Kotlin stack
- `docs/stack-rules/nodejs-typescript-rules.md` - Specific rules for Node.js/TypeScript stack
- `docs/metrics/tracking-guide.md` - Guide for tracking review metrics
- `docs/metrics/data-structure.md` - Data structure specification for metrics collection
- `README.md` - Main project documentation and quick start guide

### Notes

- All files must be written in English
- Templates should specify that Copilot can respond in Spanish (es-ES) while instructions remain in English
- Follow Markdown best practices for readability
- Use clear hierarchical structure with headers and lists

## Tasks

- [ ] 1.0 Crear estructura base de directorios y archivos de configuración
  - [x] 1.1 Create `.github/` directory if it doesn't exist
  - [x] 1.2 Create `docs/review/` directory structure
  - [x] 1.3 Create `docs/stack-rules/` directory for stack-specific rules
  - [x] 1.4 Create `docs/metrics/` directory for metrics documentation
  - [x] 1.5 Create `templates/` directory for customizable templates
  - [x] 1.6 Create `.gitignore` entries if needed for generated or temporary files

- [ ] 2.0 Implementar instrucciones principales de Copilot (.github/copilot-instructions.md)
  - [ ] 2.1 Create file header with purpose and scope section
  - [ ] 2.2 Define response style section (language: es-ES, professional tone, severity levels)
  - [ ] 2.3 Document output format structure (summary, findings with severity, quick checklist)
  - [ ] 2.4 Specify review priorities (Security > Reliability > Performance > Testing > Observability > Readability)
  - [ ] 2.5 Add cross-cutting rules section with references to specialized checklists
  - [ ] 2.6 Define severity levels (Bloqueante, Importante, Sugerencia) with clear criteria
  - [ ] 2.7 Add exclusion patterns section (generated files, lockfiles, non-technical markdown)
  - [ ] 2.8 Include output template with examples of good feedback
  - [ ] 2.9 Add metadata section (owners, last review date, how to propose changes)

- [ ] 3.0 Crear checklists especializados por área técnica (security, testing, performance, reliability, readability)
  - [ ] 3.1 Create `docs/review/security-checklist.md` with comprehensive security validations
    - [ ] 3.1.1 Add secrets and credentials checks (no plaintext, use Key Vault/KMS)
    - [ ] 3.1.2 Add input/output validation rules (XSS, SQLi/NoSQLi, SSRF prevention)
    - [ ] 3.1.3 Add authentication and authorization checks (token expiration, minimal scopes)
    - [ ] 3.1.4 Add dependency vulnerability checks (CVEs, suggest safe versions)
    - [ ] 3.1.5 Add encryption requirements (TLS in transit, encryption at rest)
    - [ ] 3.1.6 Add PII/secret masking requirements for logs and traces
  - [ ] 3.2 Create `docs/review/testing-checklist.md` with testing requirements
    - [ ] 3.2.1 Add unit test requirements for new/critical logic
    - [ ] 3.2.2 Add contract/API test requirements when schemas change
    - [ ] 3.2.3 Add test data requirements (realistic, no real PII)
    - [ ] 3.2.4 Add coverage criteria or rationale if not applicable
  - [ ] 3.3 Create `docs/review/performance-checklist.md` with optimization rules
    - [ ] 3.3.1 Add N+1 query prevention and index requirements
    - [ ] 3.3.2 Add pagination/streaming requirements for large responses
    - [ ] 3.3.3 Add caching rules with TTL and invalidation strategies
    - [ ] 3.3.4 Add complexity requirements (avoid unnecessary O(n²))
  - [ ] 3.4 Create `docs/review/reliability-checklist.md` with resilience patterns
    - [ ] 3.4.1 Add timeout, retry with backoff, and circuit breaker requirements
    - [ ] 3.4.2 Add idempotency requirements for operations with side effects
    - [ ] 3.4.3 Add explicit error handling and partial failure management
  - [ ] 3.5 Create `docs/review/readability-checklist.md` with code quality standards
    - [ ] 3.5.1 Add rules against nested ternaries
    - [ ] 3.5.2 Add cyclomatic complexity guidelines
    - [ ] 3.5.3 Add naming convention requirements (expressive names)
    - [ ] 3.5.4 Add comment guidelines (only when adding context)
    - [ ] 3.5.5 Add type/contract visibility requirements

- [ ] 4.0 Implementar reglas específicas por stack tecnológico (Java/Kotlin y Node.js/TypeScript)
  - [ ] 4.1 Create `docs/stack-rules/java-kotlin-rules.md`
    - [ ] 4.1.1 Add null-safety requirements and best practices
    - [ ] 4.1.2 Add sealed/data classes usage guidelines
    - [ ] 4.1.3 Add repository/DAO rules (parameterized queries, required indexes)
    - [ ] 4.1.4 Add dependency injection and configuration best practices
    - [ ] 4.1.5 Add logging and exception handling patterns
  - [ ] 4.2 Create `docs/stack-rules/nodejs-typescript-rules.md`
    - [ ] 4.2.1 Add ESLint/TSConfig requirements (avoid `any` unless justified)
    - [ ] 4.2.2 Add Promise handling rules (safe await, consistent error handling)
    - [ ] 4.2.3 Add HTTP library requirements (timeouts, abort controllers)
    - [ ] 4.2.4 Add async/await best practices
    - [ ] 4.2.5 Add module system and import/export conventions
  - [ ] 4.3 Add extensibility notes for future stack additions (Python, Swift, etc.)

- [ ] 5.0 Crear plantillas personalizables y guía de uso para equipos
  - [ ] 5.1 Create `templates/copilot-instructions-template.md` as customizable base
    - [ ] 5.1.1 Include all sections from main instructions with placeholder comments
    - [ ] 5.1.2 Mark clearly customizable sections (team-specific rules, exclusions)
    - [ ] 5.1.3 Add inline comments explaining each section's purpose
    - [ ] 5.1.4 Include examples of common customizations
  - [ ] 5.2 Create `templates/team-customization-guide.md` with step-by-step instructions
    - [ ] 5.2.1 Explain how to copy and customize the base template
    - [ ] 5.2.2 Document common customization scenarios (different severity levels, additional stacks)
    - [ ] 5.2.3 Provide examples of team-specific rules
    - [ ] 5.2.4 Add troubleshooting section for common issues
    - [ ] 5.2.5 Include best practices for maintaining customized versions
  - [ ] 5.3 Update main README.md with quick start guide
    - [ ] 5.3.1 Add "Getting Started" section
    - [ ] 5.3.2 Add "For Teams" section with customization instructions
    - [ ] 5.3.3 Add "Contributing" section for proposing changes
    - [ ] 5.3.4 Add examples and use cases

- [ ] 6.0 Documentar sistema de métricas y preparar estructura para tracking
  - [ ] 6.1 Create `docs/metrics/tracking-guide.md` with metrics overview
    - [ ] 6.1.1 Document Phase 1 basic metrics (adoption rate, common issue types)
    - [ ] 6.1.2 Document Phase 2 intermediate metrics (review time reduction, false positive rate)
    - [ ] 6.1.3 Document Phase 3 advanced metrics (correlation with production bugs, ROI)
    - [ ] 6.1.4 Explain how to collect metrics manually initially
  - [ ] 6.2 Create `docs/metrics/data-structure.md` with data schema specification
    - [ ] 6.2.1 Define required fields in Copilot responses for tracking
    - [ ] 6.2.2 Specify data format for metric collection (JSON schema)
    - [ ] 6.2.3 Document how to structure severity, area, and resolution data
    - [ ] 6.2.4 Add examples of properly formatted metric data
  - [ ] 6.3 Add notes on future dashboard integration points
  - [ ] 6.4 Document feedback collection process for continuous improvement