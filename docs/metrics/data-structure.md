# Metrics Data Structure Specification

This document defines the data structures and JSON schemas for collecting, storing, and reporting metrics from the GitHub Copilot PR review system.

---

## Table of Contents

1. [Overview](#overview)
2. [Data Collection Points](#data-collection-points)
3. [Core Data Structures](#core-data-structures)
4. [JSON Schema Definitions](#json-schema-definitions)
5. [Example Data](#example-data)
6. [Storage Recommendations](#storage-recommendations)
7. [Data Validation](#data-validation)
8. [API Endpoints (Future)](#api-endpoints-future)

---

## Overview

### Design Principles

- **Structured and typed** - Use JSON with clear schemas
- **Extensible** - Easy to add new fields without breaking existing data
- **Normalized** - Minimize redundancy while maintaining queryability
- **Privacy-aware** - No PII, sensitive code snippets, or credentials
- **Timestamped** - All records include ISO 8601 timestamps
- **Versioned** - Schema version for future migrations

### Data Hierarchy

```
MetricsCollection
├── Weekly Metrics
│   ├── Adoption Metrics
│   └── Aggregated Statistics
├── PR Review Records
│   ├── Findings
│   └── Resolutions
├── Team Feedback
│   └── Survey Responses
└── Incident Correlations
    └── Production Bug Analysis
```

---

## Data Collection Points

### 1. Weekly Aggregated Metrics
- **When**: End of each week
- **Source**: Manual count or GitHub API
- **Purpose**: Track adoption and high-level trends

### 2. Per-PR Review Data
- **When**: After each PR with Copilot review
- **Source**: Parse PR comments or manual entry
- **Purpose**: Detailed analysis of findings and resolutions

### 3. Team Feedback
- **When**: Weekly or monthly surveys
- **Source**: Google Forms, Slack polls, or direct feedback
- **Purpose**: Qualitative assessment and improvement ideas

### 4. Incident Correlation
- **When**: Post-incident retrospectives
- **Source**: Incident management system
- **Purpose**: Calculate bug prevention rate and cost avoidance

---

## Core Data Structures

### 1. Weekly Metrics Record

**Purpose**: Track overall adoption and activity for a given week

**Required fields**:
- `week` - ISO week identifier (e.g., "2025-W01")
- `repository` - Repository name
- `totalPRs` - Total pull requests created
- `prsWithCopilotReview` - PRs that received Copilot review
- `adoptionRate` - Percentage (calculated)

**Optional fields**:
- `teamName` - Team or squad name
- `totalFindings` - Sum of all findings (B + I + S)
- `blockingFindings` - Count of blocking issues
- `importantFindings` - Count of important issues
- `suggestionFindings` - Count of suggestions
- `falsePositiveCount` - Number of false positives reported
- `falsePositiveRate` - Percentage (calculated)
- `averageFindingsPerPR` - Average findings per reviewed PR

**Example**:
```json
{
  "schemaVersion": "1.0",
  "week": "2025-W15",
  "repository": "solo-user-service",
  "teamName": "User Management",
  "totalPRs": 32,
  "prsWithCopilotReview": 24,
  "adoptionRate": 75.0,
  "totalFindings": 145,
  "blockingFindings": 8,
  "importantFindings": 42,
  "suggestionFindings": 95,
  "falsePositiveCount": 12,
  "falsePositiveRate": 8.3,
  "averageFindingsPerPR": 6.04
}
```

---

### 2. PR Review Record

**Purpose**: Detailed record of a single PR review

**Required fields**:
- `prNumber` - Pull request number
- `repository` - Repository name
- `reviewDate` - ISO 8601 timestamp
- `findings` - Array of finding objects

**Optional fields**:
- `prTitle` - Title of the PR
- `prAuthor` - GitHub username (anonymized if needed)
- `linesChanged` - Total lines added + removed
- `filesChanged` - Number of files modified
- `reviewTimeHuman` - Human review time in minutes
- `reviewTimeCopilot` - Copilot review time in minutes (typically 1-3)
- `timeSaved` - Difference in minutes

**Example**:
```json
{
  "schemaVersion": "1.0",
  "prNumber": 1234,
  "repository": "solo-rating-service",
  "reviewDate": "2025-04-15T14:30:00Z",
  "prTitle": "Add payment validation logic",
  "prAuthor": "developer-42",
  "linesChanged": 245,
  "filesChanged": 8,
  "reviewTimeHuman": 22,
  "reviewTimeCopilot": 2,
  "timeSaved": 20,
  "findings": [
    {
      "id": "finding-1234-001",
      "severity": "blocking",
      "category": "security",
      "area": "Input Validation",
      "description": "No input validation for payment amount",
      "evidence": "PaymentService.kt:45",
      "proposedFix": "Add validation: require(amount > 0 && amount < MAX_AMOUNT)",
      "reference": "docs/review/security-checklist.md#input-output-validation",
      "resolution": "fixed",
      "resolutionTime": 15,
      "resolutionNotes": "Added validation with unit tests"
    },
    {
      "id": "finding-1234-002",
      "severity": "important",
      "category": "testing",
      "area": "Unit Tests",
      "description": "Missing tests for edge cases",
      "evidence": "PaymentServiceTest.kt",
      "proposedFix": "Add tests for: zero amount, negative amount, amount > max",
      "reference": "docs/review/testing-checklist.md#unit-tests",
      "resolution": "fixed",
      "resolutionTime": 25,
      "resolutionNotes": "Added 5 edge case tests"
    },
    {
      "id": "finding-1234-003",
      "severity": "suggestion",
      "category": "readability",
      "area": "Naming",
      "description": "Variable name 'amt' should be 'amount'",
      "evidence": "PaymentService.kt:52",
      "proposedFix": "Rename 'amt' to 'amount' for clarity",
      "reference": "docs/review/readability-checklist.md#naming",
      "resolution": "wontfix",
      "resolutionNotes": "Team convention uses 'amt' in payment context"
    },
    {
      "id": "finding-1234-004",
      "severity": "suggestion",
      "category": "performance",
      "area": "Database",
      "description": "Consider adding index on payment_status column",
      "evidence": "PaymentEntity.kt:15",
      "proposedFix": "Add @Index annotation for status column",
      "reference": "docs/review/performance-checklist.md#database-indexes",
      "resolution": "false-positive",
      "resolutionNotes": "Index already exists in migration V005"
    }
  ],
  "summary": {
    "totalFindings": 4,
    "byCategory": {
      "security": 1,
      "testing": 1,
      "readability": 1,
      "performance": 1
    },
    "bySeverity": {
      "blocking": 1,
      "important": 1,
      "suggestion": 2
    },
    "byResolution": {
      "fixed": 2,
      "wontfix": 1,
      "falsePositive": 1
    }
  }
}
```

---

### 3. Finding Object

**Purpose**: Individual issue found by Copilot

**Required fields**:
- `id` - Unique identifier (e.g., "finding-{prNumber}-{sequence}")
- `severity` - One of: `"blocking"`, `"important"`, `"suggestion"`
- `category` - One of: `"security"`, `"testing"`, `"performance"`, `"reliability"`, `"readability"`, `"code-conventions"`, `"other"`
- `description` - Brief description of the issue
- `resolution` - One of: `"fixed"`, `"wontfix"`, `"false-positive"`, `"pending"`

**Optional fields**:
- `area` - Subcategory (e.g., "Input Validation", "Unit Tests", "Database")
- `evidence` - File and line reference
- `proposedFix` - Suggested solution
- `reference` - Link to checklist or documentation
- `resolutionTime` - Minutes to resolve
- `resolutionNotes` - Explanation of resolution

**Enums and allowed values**:

```typescript
// Severity levels
type Severity = "blocking" | "important" | "suggestion";

// Categories (aligned with review areas)
type Category = 
  | "security"
  | "testing"
  | "performance"
  | "reliability"
  | "readability"
  | "code-conventions"
  | "other";

// Resolution status
type Resolution = "fixed" | "wontfix" | "false-positive" | "pending";

// Common areas by category
type Area = 
  // Security
  | "Secrets Management"
  | "Input Validation"
  | "Authentication"
  | "Encryption"
  | "PII Handling"
  // Testing
  | "Unit Tests"
  | "Integration Tests"
  | "Test Data"
  | "Coverage"
  // Performance
  | "Database Queries"
  | "Caching"
  | "Algorithmic Complexity"
  | "Network I/O"
  // Reliability
  | "Error Handling"
  | "Timeouts"
  | "Retries"
  | "Circuit Breakers"
  // Readability
  | "Naming"
  | "Complexity"
  | "Comments"
  | "Code Organization"
  // Code Conventions
  | "Team Standards"
  | "Stack-Specific Rules"
  | string; // Allow custom areas
```

**Example finding**:
```json
{
  "id": "finding-5678-003",
  "severity": "blocking",
  "category": "security",
  "area": "Secrets Management",
  "description": "Hardcoded database password in configuration file",
  "evidence": "application.properties:15",
  "proposedFix": "Move to environment variable or AWS Secrets Manager",
  "reference": "docs/review/security-checklist.md#secrets-management",
  "resolution": "fixed",
  "resolutionTime": 30,
  "resolutionNotes": "Migrated to AWS Secrets Manager with environment variable DB_SECRET_ARN"
}
```

---

### 4. Team Feedback Record

**Purpose**: Capture qualitative feedback from developers

**Required fields**:
- `week` - ISO week identifier
- `repository` - Repository name
- `respondent` - Anonymous ID or team name
- `helpfulnessScore` - Rating 1-5
- `timestamp` - ISO 8601 timestamp

**Optional fields**:
- `falsePositiveCount` - Number of false positives encountered
- `missedIssuesCount` - Number of issues Copilot missed
- `wouldRecommend` - Boolean
- `mostValuableAspect` - Free text
- `improvementSuggestions` - Free text
- `specificFalsePositives` - Array of examples

**Example**:
```json
{
  "schemaVersion": "1.0",
  "week": "2025-W15",
  "repository": "solo-payment-service",
  "respondent": "team-payments",
  "timestamp": "2025-04-18T16:45:00Z",
  "helpfulnessScore": 5,
  "falsePositiveCount": 2,
  "missedIssuesCount": 0,
  "wouldRecommend": true,
  "mostValuableAspect": "Catches missing tests and security issues early",
  "improvementSuggestions": "Reduce false positives for generated DTO classes",
  "specificFalsePositives": [
    "Flagged auto-generated Swagger client code for complexity",
    "Reported missing tests for simple data class getters"
  ]
}
```

---

### 5. Incident Correlation Record

**Purpose**: Link production incidents to Copilot capabilities

**Required fields**:
- `incidentId` - Incident tracking ID
- `incidentDate` - ISO 8601 timestamp
- `severity` - Incident severity (e.g., "critical", "high", "medium", "low")
- `category` - Type of incident (aligned with review categories)
- `copilotWouldCatch` - Boolean
- `preventable` - Boolean

**Optional fields**:
- `description` - Brief incident summary (no sensitive data)
- `rootCause` - Technical root cause
- `relevantChecklist` - Link to checklist that covers this
- `costAvoidanceHours` - Estimated hours saved if caught early
- `costAvoidanceDollars` - Estimated dollar value
- `lessonsLearned` - What to improve in instructions

**Example**:
```json
{
  "schemaVersion": "1.0",
  "incidentId": "INC-2025-089",
  "incidentDate": "2025-04-20T03:15:00Z",
  "severity": "high",
  "category": "security",
  "description": "SQL injection in user search endpoint",
  "rootCause": "String concatenation used instead of parameterized query",
  "copilotWouldCatch": true,
  "preventable": true,
  "relevantChecklist": "docs/review/security-checklist.md#sql-injection",
  "costAvoidanceHours": 24,
  "costAvoidanceDollars": 2400,
  "lessonsLearned": "Add more examples of SQL injection patterns in Kotlin"
}
```

---

## JSON Schema Definitions

### Weekly Metrics Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Weekly Metrics Record",
  "required": ["schemaVersion", "week", "repository", "totalPRs", "prsWithCopilotReview", "adoptionRate"],
  "properties": {
    "schemaVersion": {
      "type": "string",
      "description": "Schema version for future migrations",
      "const": "1.0"
    },
    "week": {
      "type": "string",
      "description": "ISO week identifier (YYYY-Www)",
      "pattern": "^\\d{4}-W\\d{2}$"
    },
    "repository": {
      "type": "string",
      "description": "Repository name"
    },
    "teamName": {
      "type": "string",
      "description": "Team or squad name"
    },
    "totalPRs": {
      "type": "integer",
      "minimum": 0
    },
    "prsWithCopilotReview": {
      "type": "integer",
      "minimum": 0
    },
    "adoptionRate": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "totalFindings": {
      "type": "integer",
      "minimum": 0
    },
    "blockingFindings": {
      "type": "integer",
      "minimum": 0
    },
    "importantFindings": {
      "type": "integer",
      "minimum": 0
    },
    "suggestionFindings": {
      "type": "integer",
      "minimum": 0
    },
    "falsePositiveCount": {
      "type": "integer",
      "minimum": 0
    },
    "falsePositiveRate": {
      "type": "number",
      "minimum": 0,
      "maximum": 100
    },
    "averageFindingsPerPR": {
      "type": "number",
      "minimum": 0
    }
  }
}
```

### PR Review Record Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "PR Review Record",
  "required": ["schemaVersion", "prNumber", "repository", "reviewDate", "findings"],
  "properties": {
    "schemaVersion": {
      "type": "string",
      "const": "1.0"
    },
    "prNumber": {
      "type": "integer",
      "minimum": 1
    },
    "repository": {
      "type": "string"
    },
    "reviewDate": {
      "type": "string",
      "format": "date-time"
    },
    "prTitle": {
      "type": "string"
    },
    "prAuthor": {
      "type": "string"
    },
    "linesChanged": {
      "type": "integer",
      "minimum": 0
    },
    "filesChanged": {
      "type": "integer",
      "minimum": 0
    },
    "reviewTimeHuman": {
      "type": "number",
      "minimum": 0
    },
    "reviewTimeCopilot": {
      "type": "number",
      "minimum": 0
    },
    "timeSaved": {
      "type": "number"
    },
    "findings": {
      "type": "array",
      "items": {
        "$ref": "#/definitions/finding"
      }
    },
    "summary": {
      "type": "object",
      "properties": {
        "totalFindings": {
          "type": "integer",
          "minimum": 0
        },
        "byCategory": {
          "type": "object",
          "additionalProperties": {
            "type": "integer",
            "minimum": 0
          }
        },
        "bySeverity": {
          "type": "object",
          "properties": {
            "blocking": { "type": "integer", "minimum": 0 },
            "important": { "type": "integer", "minimum": 0 },
            "suggestion": { "type": "integer", "minimum": 0 }
          }
        },
        "byResolution": {
          "type": "object",
          "properties": {
            "fixed": { "type": "integer", "minimum": 0 },
            "wontfix": { "type": "integer", "minimum": 0 },
            "falsePositive": { "type": "integer", "minimum": 0 },
            "pending": { "type": "integer", "minimum": 0 }
          }
        }
      }
    }
  },
  "definitions": {
    "finding": {
      "type": "object",
      "required": ["id", "severity", "category", "description", "resolution"],
      "properties": {
        "id": {
          "type": "string"
        },
        "severity": {
          "type": "string",
          "enum": ["blocking", "important", "suggestion"]
        },
        "category": {
          "type": "string",
          "enum": ["security", "testing", "performance", "reliability", "readability", "code-conventions", "other"]
        },
        "area": {
          "type": "string"
        },
        "description": {
          "type": "string"
        },
        "evidence": {
          "type": "string"
        },
        "proposedFix": {
          "type": "string"
        },
        "reference": {
          "type": "string"
        },
        "resolution": {
          "type": "string",
          "enum": ["fixed", "wontfix", "false-positive", "pending"]
        },
        "resolutionTime": {
          "type": "number",
          "minimum": 0
        },
        "resolutionNotes": {
          "type": "string"
        }
      }
    }
  }
}
```

### Team Feedback Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "title": "Team Feedback Record",
  "required": ["schemaVersion", "week", "repository", "respondent", "helpfulnessScore", "timestamp"],
  "properties": {
    "schemaVersion": {
      "type": "string",
      "const": "1.0"
    },
    "week": {
      "type": "string",
      "pattern": "^\\d{4}-W\\d{2}$"
    },
    "repository": {
      "type": "string"
    },
    "respondent": {
      "type": "string"
    },
    "timestamp": {
      "type": "string",
      "format": "date-time"
    },
    "helpfulnessScore": {
      "type": "integer",
      "minimum": 1,
      "maximum": 5
    },
    "falsePositiveCount": {
      "type": "integer",
      "minimum": 0
    },
    "missedIssuesCount": {
      "type": "integer",
      "minimum": 0
    },
    "wouldRecommend": {
      "type": "boolean"
    },
    "mostValuableAspect": {
      "type": "string"
    },
    "improvementSuggestions": {
      "type": "string"
    },
    "specificFalsePositives": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }
  }
}
```

---

## Example Data

### Complete Weekly Metrics Collection

```json
{
  "collectionDate": "2025-04-21T23:59:59Z",
  "weeklyMetrics": [
    {
      "schemaVersion": "1.0",
      "week": "2025-W16",
      "repository": "solo-user-service",
      "teamName": "User Management",
      "totalPRs": 28,
      "prsWithCopilotReview": 22,
      "adoptionRate": 78.6,
      "totalFindings": 132,
      "blockingFindings": 6,
      "importantFindings": 38,
      "suggestionFindings": 88,
      "falsePositiveCount": 11,
      "falsePositiveRate": 8.3,
      "averageFindingsPerPR": 6.0
    },
    {
      "schemaVersion": "1.0",
      "week": "2025-W16",
      "repository": "solo-rating-service",
      "teamName": "Rating & Reviews",
      "totalPRs": 15,
      "prsWithCopilotReview": 13,
      "adoptionRate": 86.7,
      "totalFindings": 78,
      "blockingFindings": 3,
      "importantFindings": 22,
      "suggestionFindings": 53,
      "falsePositiveCount": 5,
      "falsePositiveRate": 6.4,
      "averageFindingsPerPR": 6.0
    }
  ]
}
```

### Sample PR Reviews Collection

```json
{
  "collectionDate": "2025-04-21T23:59:59Z",
  "prReviews": [
    {
      "schemaVersion": "1.0",
      "prNumber": 8901,
      "repository": "solo-user-service",
      "reviewDate": "2025-04-18T10:30:00Z",
      "prTitle": "Add user profile image upload",
      "prAuthor": "dev-alice",
      "linesChanged": 342,
      "filesChanged": 12,
      "reviewTimeHuman": 28,
      "reviewTimeCopilot": 2,
      "timeSaved": 26,
      "findings": [
        {
          "id": "finding-8901-001",
          "severity": "blocking",
          "category": "security",
          "area": "Input Validation",
          "description": "No file type validation for image uploads",
          "evidence": "ImageUploadController.kt:67",
          "proposedFix": "Validate MIME type and file extension, allow only image/* types",
          "reference": "docs/review/security-checklist.md#input-output-validation",
          "resolution": "fixed",
          "resolutionTime": 20,
          "resolutionNotes": "Added MIME type validation with whitelist"
        },
        {
          "id": "finding-8901-002",
          "severity": "blocking",
          "category": "security",
          "area": "Input Validation",
          "description": "No file size limit on uploads",
          "evidence": "ImageUploadController.kt:70",
          "proposedFix": "Add max file size limit (e.g., 5MB) to prevent DoS",
          "reference": "docs/review/security-checklist.md#file-upload-security",
          "resolution": "fixed",
          "resolutionTime": 10,
          "resolutionNotes": "Added 5MB size limit with configuration"
        },
        {
          "id": "finding-8901-003",
          "severity": "important",
          "category": "testing",
          "area": "Unit Tests",
          "description": "Missing tests for upload error scenarios",
          "evidence": "ImageUploadControllerTest.kt",
          "proposedFix": "Add tests for: invalid file type, file too large, upload failure",
          "reference": "docs/review/testing-checklist.md#unit-tests",
          "resolution": "fixed",
          "resolutionTime": 35,
          "resolutionNotes": "Added 6 error scenario tests"
        },
        {
          "id": "finding-8901-004",
          "severity": "important",
          "category": "performance",
          "area": "Caching",
          "description": "Uploaded images should be cached with CDN",
          "evidence": "ImageStorageService.kt:45",
          "proposedFix": "Add CloudFront CDN in front of S3 bucket",
          "reference": "docs/review/performance-checklist.md#caching",
          "resolution": "wontfix",
          "resolutionNotes": "CDN planned for Q3, tracked in INFRA-234"
        },
        {
          "id": "finding-8901-005",
          "severity": "suggestion",
          "category": "readability",
          "area": "Naming",
          "description": "Function name 'uploadImg' should be 'uploadImage'",
          "evidence": "ImageService.kt:28",
          "proposedFix": "Rename for clarity and consistency",
          "reference": "docs/review/readability-checklist.md#naming",
          "resolution": "fixed",
          "resolutionTime": 2,
          "resolutionNotes": "Renamed function and updated all callers"
        }
      ],
      "summary": {
        "totalFindings": 5,
        "byCategory": {
          "security": 2,
          "testing": 1,
          "performance": 1,
          "readability": 1
        },
        "bySeverity": {
          "blocking": 2,
          "important": 2,
          "suggestion": 1
        },
        "byResolution": {
          "fixed": 4,
          "wontfix": 1,
          "falsePositive": 0
        }
      }
    }
  ]
}
```

---

## Storage Recommendations

### Development / Pilot Phase

**Local JSON files**:
```
metrics/
├── weekly/
│   ├── 2025-W15.json
│   ├── 2025-W16.json
│   └── 2025-W17.json
├── pr-reviews/
│   ├── solo-user-service/
│   │   ├── pr-8901.json
│   │   └── pr-8902.json
│   └── solo-rating-service/
│       └── pr-5432.json
├── feedback/
│   └── 2025-W15-feedback.json
└── incidents/
    └── incidents-2025-Q2.json
```

**Pros**: Simple, no infrastructure needed, version control friendly  
**Cons**: Limited querying, no concurrent access, manual aggregation

### Production Phase

**Database options**:

1. **PostgreSQL / MySQL**
   - Tables: `weekly_metrics`, `pr_reviews`, `findings`, `team_feedback`, `incidents`
   - Pros: Strong typing, ACID compliance, complex queries
   - Cons: Requires database setup and maintenance

2. **BigQuery**
   - Partitioned by date, optimized for analytics
   - Pros: Scales well, integrated with Google Cloud, great for dashboards
   - Cons: Google Cloud dependency, query costs

3. **MongoDB / DocumentDB**
   - Store JSON documents directly
   - Pros: Schema flexibility, horizontal scaling
   - Cons: Less structure enforcement, eventual consistency

**Recommendation**: Start with JSON files, migrate to PostgreSQL when you have 10+ weeks of data or need dashboards.

---

## Data Validation

### Pre-Storage Validation

**Required validations**:
- Schema version present and valid
- Required fields not null/empty
- Enums match allowed values (severity, category, resolution)
- Numeric fields within expected ranges
- Dates in ISO 8601 format
- No PII or sensitive data

**Validation script example** (Python):
```python
import json
import jsonschema
from datetime import datetime

def validate_pr_review(data):
    # Load schema
    with open('schemas/pr-review-schema.json') as f:
        schema = json.load(f)
    
    # Validate against schema
    jsonschema.validate(instance=data, schema=schema)
    
    # Additional business logic validations
    assert data['prsWithCopilotReview'] <= data['totalPRs'], \
        "Reviewed PRs cannot exceed total PRs"
    
    assert 0 <= data['adoptionRate'] <= 100, \
        "Adoption rate must be 0-100"
    
    # Check for PII (basic)
    sensitive_patterns = ['password', 'secret', 'api_key', '@gmail.com', '@.*\\.com']
    for pattern in sensitive_patterns:
        assert not re.search(pattern, json.dumps(data), re.IGNORECASE), \
            f"Potential PII detected: {pattern}"
    
    print("✓ Validation passed")
```

### Data Quality Checks

**Monthly quality audit**:
- Check for missing weeks or gaps
- Verify adoption rate trends (sudden drops = issue)
- Identify outliers (e.g., 50 findings in one PR = review issue)
- Validate resolution rates (100% false positives = bad rules)

---

## API Endpoints (Future)

### Proposed REST API

**POST /api/metrics/weekly**
```json
{
  "week": "2025-W16",
  "repository": "solo-user-service",
  "totalPRs": 28,
  "prsWithCopilotReview": 22
}
```

**POST /api/metrics/pr-reviews**
```json
{
  "prNumber": 8901,
  "repository": "solo-user-service",
  "findings": [...]
}
```

**GET /api/metrics/weekly?repository={repo}&start={week}&end={week}**
```json
{
  "data": [...]
}
```

**GET /api/metrics/summary?period=month&repository={repo}**
```json
{
  "adoptionRate": 78.5,
  "totalFindings": 450,
  "falsePositiveRate": 7.8,
  "timeSaved": 240
}
```

---

## Migration and Versioning

### Schema Evolution

**When to increment schema version**:
- Add required fields (breaking change → v2.0)
- Change field types (breaking → v2.0)
- Add optional fields (non-breaking → v1.1)
- Deprecate fields (non-breaking → v1.1, remove in v2.0)

**Version strategy**:
```json
{
  "schemaVersion": "1.0",
  "schemaDeprecations": [],
  ...
}
```

**Migration process**:
1. Announce schema change with 1-month notice
2. Support both old and new schemas during transition
3. Provide migration script for existing data
4. Deprecate old schema after 3 months
5. Remove support after 6 months

---

## Resources

- [Tracking Guide](tracking-guide.md) - How to collect metrics
- [JSON Schema Documentation](https://json-schema.org/)
- [ISO 8601 Date Format](https://en.wikipedia.org/wiki/ISO_8601)

---

## Questions or Feedback?

For questions about data structures or schema changes, please:
- Open an issue in the repository
- Contact VW D:H
