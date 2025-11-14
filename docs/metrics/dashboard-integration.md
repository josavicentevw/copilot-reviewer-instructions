# Dashboard Integration Notes

This document outlines future integration points for visualizing metrics from the GitHub Copilot PR review system using dashboard and analytics tools.

---

## Table of Contents

1. [Overview](#overview)
2. [Integration Architecture](#integration-architecture)
3. [BigQuery Integration](#bigquery-integration)
4. [Grafana Dashboard](#grafana-dashboard)
5. [Looker / Data Studio](#looker--data-studio)
6. [GitHub Actions Automation](#github-actions-automation)
7. [Alerting and Notifications](#alerting-and-notifications)
8. [Implementation Roadmap](#implementation-roadmap)

---

## Overview

### Current State (Manual Collection)

- Metrics collected manually via spreadsheets or JSON files
- Weekly reporting via markdown documents
- No real-time visualization
- Limited historical trend analysis

### Future State (Automated Dashboards)

- Automated data collection from PR comments
- Real-time metrics dashboards
- Historical trend analysis
- Alerting on anomalies (e.g., sudden spike in false positives)
- Executive-level reporting with ROI metrics

### Goals

- **Reduce manual effort** - Automate collection and reporting
- **Increase visibility** - Make metrics accessible to all stakeholders
- **Enable insights** - Identify patterns and improvement opportunities
- **Demonstrate value** - Show ROI and impact on code quality

---

## Integration Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    GitHub Pull Requests                      │
│                  (with Copilot reviews)                      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ GitHub Webhooks
                     │ or scheduled GitHub Actions
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  Metrics Collection Service                  │
│  - Parse Copilot PR comments                                │
│  - Extract findings (severity, category, evidence)          │
│  - Validate and structure data                              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ Insert/Update
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    Data Warehouse                            │
│              (BigQuery / PostgreSQL / MongoDB)               │
│  Tables:                                                     │
│  - weekly_metrics                                            │
│  - pr_reviews                                                │
│  - findings                                                  │
│  - team_feedback                                             │
│  - incidents                                                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     │ Query / API
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   Dashboard Layer                            │
│  - Grafana: Real-time operational metrics                   │
│  - Looker/Data Studio: Executive reports                    │
│  - Custom web app: Team-specific views                      │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **PR Created/Updated** → GitHub webhook triggers
2. **Copilot Reviews PR** → Comment posted with findings
3. **Collector Service** → Parses comment, extracts structured data
4. **Validation** → Ensures data quality and schema compliance
5. **Storage** → Inserts into data warehouse
6. **Dashboards** → Query data warehouse and visualize
7. **Alerts** → Monitor for anomalies and notify teams

---

## BigQuery Integration

### Why BigQuery?

- **Scalable** - Handles large volumes of metrics data
- **Analytics-optimized** - Fast aggregation and querying
- **Cost-effective** - Pay only for queries and storage
- **Google Cloud integration** - Works with Data Studio, Cloud Functions
- **SQL interface** - Familiar query language

### Schema Design

**Table: `weekly_metrics`**
```sql
CREATE TABLE copilot_metrics.weekly_metrics (
  week STRING NOT NULL,                    -- ISO week (2025-W16)
  repository STRING NOT NULL,
  team_name STRING,
  total_prs INT64,
  prs_with_copilot_review INT64,
  adoption_rate FLOAT64,
  total_findings INT64,
  blocking_findings INT64,
  important_findings INT64,
  suggestion_findings INT64,
  false_positive_count INT64,
  false_positive_rate FLOAT64,
  average_findings_per_pr FLOAT64,
  ingestion_timestamp TIMESTAMP,
  PRIMARY KEY (week, repository) NOT ENFORCED
)
PARTITION BY DATE(_PARTITIONTIME)
CLUSTER BY repository, week;
```

**Table: `pr_reviews`**
```sql
CREATE TABLE copilot_metrics.pr_reviews (
  pr_number INT64 NOT NULL,
  repository STRING NOT NULL,
  review_date TIMESTAMP,
  pr_title STRING,
  pr_author STRING,
  lines_changed INT64,
  files_changed INT64,
  review_time_human FLOAT64,
  review_time_copilot FLOAT64,
  time_saved FLOAT64,
  total_findings INT64,
  blocking_findings INT64,
  important_findings INT64,
  suggestion_findings INT64,
  ingestion_timestamp TIMESTAMP,
  PRIMARY KEY (pr_number, repository) NOT ENFORCED
)
PARTITION BY DATE(review_date)
CLUSTER BY repository, review_date;
```

**Table: `findings`**
```sql
CREATE TABLE copilot_metrics.findings (
  finding_id STRING NOT NULL,              -- finding-{prNumber}-{sequence}
  pr_number INT64,
  repository STRING,
  review_date TIMESTAMP,
  severity STRING,                          -- blocking, important, suggestion
  category STRING,                          -- security, testing, performance, etc.
  area STRING,
  description STRING,
  evidence STRING,
  proposed_fix STRING,
  reference STRING,
  resolution STRING,                        -- fixed, wontfix, false-positive
  resolution_time FLOAT64,
  resolution_notes STRING,
  ingestion_timestamp TIMESTAMP,
  PRIMARY KEY (finding_id) NOT ENFORCED
)
PARTITION BY DATE(review_date)
CLUSTER BY category, severity, repository;
```

### Sample Queries

**Adoption rate over time**:
```sql
SELECT 
  week,
  AVG(adoption_rate) as avg_adoption_rate,
  SUM(total_prs) as total_prs,
  SUM(prs_with_copilot_review) as reviewed_prs
FROM copilot_metrics.weekly_metrics
WHERE week >= '2025-W01'
GROUP BY week
ORDER BY week;
```

**Top issue categories**:
```sql
SELECT 
  category,
  COUNT(*) as finding_count,
  ROUND(COUNT(*) * 100.0 / SUM(COUNT(*)) OVER(), 2) as percentage
FROM copilot_metrics.findings
WHERE review_date >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY category
ORDER BY finding_count DESC;
```

**False positive rate by repository**:
```sql
SELECT 
  repository,
  COUNT(*) as total_findings,
  COUNTIF(resolution = 'false-positive') as false_positives,
  ROUND(COUNTIF(resolution = 'false-positive') * 100.0 / COUNT(*), 2) as fp_rate
FROM copilot_metrics.findings
WHERE review_date >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 90 DAY)
GROUP BY repository
HAVING total_findings > 10
ORDER BY fp_rate DESC;
```

**Time savings calculation**:
```sql
SELECT 
  DATE_TRUNC(review_date, MONTH) as month,
  SUM(time_saved) as total_minutes_saved,
  ROUND(SUM(time_saved) / 60, 2) as total_hours_saved,
  ROUND(SUM(time_saved) / 60 * 75, 2) as cost_savings_usd  -- $75/hour
FROM copilot_metrics.pr_reviews
WHERE review_date >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 12 MONTH)
GROUP BY month
ORDER BY month;
```

### Data Ingestion

**Option 1: Cloud Function triggered by GitHub webhook**
```python
# cloud_function.py
import json
from google.cloud import bigquery

def ingest_pr_review(request):
    """Parse GitHub PR comment and insert into BigQuery"""
    payload = request.get_json()
    
    if payload.get('action') != 'created':
        return {'status': 'ignored'}
    
    # Parse Copilot review comment
    comment = payload.get('comment', {}).get('body', '')
    if 'GitHub Copilot' not in comment:
        return {'status': 'not_copilot_review'}
    
    # Extract structured data from comment
    review_data = parse_copilot_comment(comment, payload)
    
    # Insert into BigQuery
    client = bigquery.Client()
    table = client.get_table('copilot_metrics.pr_reviews')
    errors = client.insert_rows_json(table, [review_data])
    
    if errors:
        return {'status': 'error', 'errors': errors}
    
    return {'status': 'success'}
```

**Option 2: GitHub Actions scheduled job**
```yaml
# .github/workflows/collect-metrics.yml
name: Collect Weekly Metrics
on:
  schedule:
    - cron: '0 0 * * 0'  # Every Sunday at midnight

jobs:
  collect:
    runs-on: ubuntu-latest
    steps:
      - name: Fetch PRs from last week
        run: |
          gh pr list --state closed --limit 100 --json number,title,createdAt
      
      - name: Parse Copilot reviews
        run: python scripts/parse_reviews.py
      
      - name: Upload to BigQuery
        run: |
          bq load --source_format=NEWLINE_DELIMITED_JSON \
            copilot_metrics.weekly_metrics \
            metrics/weekly-$(date +%Y-W%V).json
```

---

## Grafana Dashboard

### Why Grafana?

- **Real-time monitoring** - Live dashboards for operational metrics
- **Alerting** - Notify teams when thresholds breached
- **Flexible** - Supports many data sources (PostgreSQL, BigQuery via plugin)
- **Open source** - Free and extensible

### Sample Dashboard Panels

**Panel 1: Adoption Rate Trend**
```
Data source: PostgreSQL or BigQuery
Query: SELECT week, AVG(adoption_rate) FROM weekly_metrics GROUP BY week
Visualization: Time series line chart
Target line: 80% adoption
```

**Panel 2: Findings by Severity**
```
Query: SELECT severity, COUNT(*) FROM findings WHERE date > now() - interval '30 days' GROUP BY severity
Visualization: Pie chart
Colors: Blocking=red, Important=orange, Suggestion=blue
```

**Panel 3: False Positive Rate**
```
Query: SELECT repository, false_positive_rate FROM weekly_metrics WHERE week = current_week
Visualization: Bar gauge
Thresholds: Green < 5%, Yellow 5-10%, Red > 10%
```

**Panel 4: Time Savings**
```
Query: SELECT DATE_TRUNC('week', review_date), SUM(time_saved)/60 FROM pr_reviews GROUP BY 1
Visualization: Bar chart
Unit: Hours
```

**Panel 5: Top Issue Categories (Last 30 Days)**
```
Query: SELECT category, COUNT(*) FROM findings WHERE date > now() - 30 GROUP BY category ORDER BY count DESC LIMIT 10
Visualization: Horizontal bar chart
```

### Alert Rules

**Alert 1: Low Adoption Rate**
```yaml
alert: LowAdoptionRate
expr: avg(adoption_rate) < 50
for: 1w
annotations:
  summary: "Copilot adoption below 50%"
  description: "Only {{ $value }}% of PRs are reviewed by Copilot"
```

**Alert 2: High False Positive Rate**
```yaml
alert: HighFalsePositiveRate
expr: false_positive_rate > 15
for: 2w
annotations:
  summary: "False positive rate > 15%"
  description: "Review instructions may need refinement"
```

**Alert 3: Spike in Blocking Issues**
```yaml
alert: BlockingIssuesSpike
expr: blocking_findings > (avg_over_time(blocking_findings[4w]) * 2)
for: 1d
annotations:
  summary: "Blocking issues 2x normal"
  description: "May indicate new pattern or code quality regression"
```

### Implementation Steps

1. Install Grafana (or use Grafana Cloud)
2. Add data source (PostgreSQL or BigQuery)
3. Import dashboard template (create from panels above)
4. Configure alert notification channels (Slack, email, PagerDuty)
5. Share dashboard link with team

---

## Looker / Data Studio

### Why Looker/Data Studio?

- **Executive reporting** - Polished, stakeholder-ready visuals
- **Scheduled reports** - Email weekly/monthly reports automatically
- **Google Sheets integration** - Easy data exploration
- **Drill-downs** - Interactive exploration from summary to details

### Sample Report Structure

**Executive Summary (Page 1)**
- **Adoption scorecard**: Current week vs target vs last quarter
- **ROI summary**: Time saved, cost avoidance, incidents prevented
- **Quality trends**: Issues decreasing over time (line chart)
- **Team satisfaction**: NPS score or average rating

**Detailed Metrics (Page 2)**
- **Findings by category** (stacked bar by week)
- **Resolution rate** (funnel chart: total → fixed → deployed)
- **False positive trends** (line chart with target threshold)
- **Top repositories** (table with adoption, findings, false positives)

**Team Deep-Dive (Page 3)**
- Filterable by team/repository
- PR velocity vs review coverage
- Common issues for this team
- Improvement recommendations

### Data Studio Dashboard Configuration

**Data source**: BigQuery connector

**Scorecard 1: Adoption Rate**
```
Metric: AVG(adoption_rate)
Date range: This week
Comparison: Previous week
Target: 80%
```

**Chart 1: Weekly Findings Trend**
```
Dimension: week
Breakdown: severity (blocking, important, suggestion)
Metric: COUNT(finding_id)
Chart type: Stacked area
```

**Chart 2: Category Distribution**
```
Dimension: category
Metric: COUNT(finding_id)
Chart type: Pie
Filter: review_date >= 30 days ago
```

**Table 1: Repository Metrics**
```
Dimensions: repository, team_name
Metrics: 
  - AVG(adoption_rate)
  - SUM(total_findings)
  - AVG(false_positive_rate)
  - SUM(time_saved) / 60 (hours)
Sort: adoption_rate DESC
```

---

## GitHub Actions Automation

### Automated Collection Workflow

**File: `.github/workflows/collect-copilot-metrics.yml`**
```yaml
name: Collect Copilot Metrics

on:
  schedule:
    # Run every Monday at 9 AM UTC
    - cron: '0 9 * * 1'
  workflow_dispatch:  # Manual trigger

jobs:
  collect-metrics:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
      
      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'
      
      - name: Install dependencies
        run: |
          pip install PyGithub google-cloud-bigquery jsonschema
      
      - name: Fetch closed PRs from last week
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          python scripts/fetch_prs.py \
            --repo ${{ github.repository }} \
            --since $(date -d '7 days ago' +%Y-%m-%d) \
            --output data/prs.json
      
      - name: Parse Copilot reviews
        run: |
          python scripts/parse_copilot_reviews.py \
            --input data/prs.json \
            --output data/reviews.json
      
      - name: Validate data
        run: |
          python scripts/validate_metrics.py \
            --input data/reviews.json \
            --schema schemas/pr-review-schema.json
      
      - name: Upload to BigQuery
        env:
          GOOGLE_APPLICATION_CREDENTIALS: ${{ secrets.GCP_SA_KEY }}
        run: |
          python scripts/upload_to_bigquery.py \
            --input data/reviews.json \
            --project copilot-metrics \
            --dataset copilot_metrics \
            --table pr_reviews
      
      - name: Generate weekly report
        run: |
          python scripts/generate_report.py \
            --week $(date +%Y-W%V) \
            --output reports/weekly-report.md
      
      - name: Post report to Slack
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
        run: |
          curl -X POST $SLACK_WEBHOOK \
            -H 'Content-Type: application/json' \
            -d @reports/weekly-report.json
```

### Parser Script Example

**File: `scripts/parse_copilot_reviews.py`**
```python
import json
import re
from github import Github

def parse_copilot_comment(comment_body):
    """Extract structured findings from Copilot review comment"""
    findings = []
    
    # Regex to match findings format:
    # - **Severity | Category**: Description
    pattern = r'-\s+\*\*(\w+)\s+\|\s+(\w+)\*\*:\s+(.+?)(?=\n\s+\*\*Evidence|$)'
    
    matches = re.finditer(pattern, comment_body, re.DOTALL)
    
    for i, match in enumerate(matches):
        severity = match.group(1).lower()
        category = match.group(2).lower()
        description = match.group(3).strip()
        
        # Extract evidence, proposed fix, reference
        evidence_match = re.search(r'\*\*Evidence:\*\*\s+`([^`]+)`', comment_body[match.end():])
        fix_match = re.search(r'\*\*Proposed fix:\*\*\s+(.+?)(?=\n\s+\*\*|$)', comment_body[match.end():])
        ref_match = re.search(r'\*\*Reference:\*\*\s+\[`([^`]+)`\]', comment_body[match.end():])
        
        finding = {
            'id': f'finding-{pr_number}-{i+1:03d}',
            'severity': severity,
            'category': category,
            'description': description,
            'evidence': evidence_match.group(1) if evidence_match else None,
            'proposedFix': fix_match.group(1).strip() if fix_match else None,
            'reference': ref_match.group(1) if ref_match else None,
            'resolution': 'pending'
        }
        findings.append(finding)
    
    return findings

# Main execution
if __name__ == '__main__':
    # Load PRs, parse comments, output JSON
    ...
```

---

## Alerting and Notifications

### Slack Integration

**Channel: `#copilot-reviews-metrics`**

**Weekly summary bot**:
```json
{
  "text": "📊 Weekly Copilot Review Summary (Week 16)",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "📊 Copilot Review Metrics - Week 16"
      }
    },
    {
      "type": "section",
      "fields": [
        { "type": "mrkdwn", "text": "*Adoption Rate:*\n78% ✅" },
        { "type": "mrkdwn", "text": "*Total Findings:*\n320" },
        { "type": "mrkdwn", "text": "*False Positive Rate:*\n7.5% ✅" },
        { "type": "mrkdwn", "text": "*Time Saved:*\n42 hours" }
      ]
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Top Issues:*\n• Missing tests (35%)\n• Security - credentials (20%)\n• Performance - N+1 queries (15%)"
      }
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": { "type": "plain_text", "text": "View Dashboard" },
          "url": "https://grafana.example.com/d/copilot-metrics"
        }
      ]
    }
  ]
}
```

**Alert notifications**:
```json
{
  "text": "⚠️ Copilot Metrics Alert: High False Positive Rate",
  "blocks": [
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "⚠️ *Alert: High False Positive Rate*\n\nRepository: `solo-user-service`\nFalse Positive Rate: 18% (threshold: 10%)\nWeek: 2025-W16"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Action Required:*\nReview and refine Copilot instructions to reduce false positives."
      }
    }
  ]
}
```

### Email Reports

**Scheduled weekly email** (via SendGrid, Mailgun, or SES):
- To: Engineering leads, platform team
- Subject: "Copilot Review Metrics - Week {week}"
- Content: HTML report with charts (embedded images)
- Attachment: CSV export for deeper analysis

---

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)
- [ ] Set up BigQuery project and tables
- [ ] Create GitHub Actions workflow for data collection
- [ ] Implement parser script for PR comments
- [ ] Test end-to-end data flow with sample data

### Phase 2: Dashboards (Weeks 5-8)
- [ ] Build Grafana dashboard with key panels
- [ ] Configure alerting rules and Slack notifications
- [ ] Create Looker/Data Studio executive report
- [ ] Share dashboard URLs with stakeholders

### Phase 3: Automation (Weeks 9-12)
- [ ] Fully automate weekly collection (no manual steps)
- [ ] Set up email report delivery
- [ ] Implement data quality monitoring
- [ ] Create runbook for troubleshooting

### Phase 4: Optimization (Months 4-6)
- [ ] Add advanced analytics (cohort analysis, predictive models)
- [ ] Integrate with incident management system
- [ ] Build team-specific dashboards
- [ ] Implement A/B testing for instruction changes

---

## Cost Estimates

### BigQuery (Google Cloud)
- **Storage**: ~1 GB/year = $0.02/month
- **Queries**: ~10 queries/day * 1 GB scanned = $0.15/month
- **Total**: ~$2/month

### Grafana Cloud (Free tier)
- 10,000 series, 14-day retention
- **Cost**: $0 (or $49/month for Pro)

### GitHub Actions
- 2,000 minutes/month free for private repos
- Metrics collection: ~10 min/week = 40 min/month
- **Cost**: $0

**Total estimated cost**: ~$2-50/month depending on tooling choices

---

## Next Steps

1. **Pilot with one repository** - Validate data collection and dashboards
2. **Iterate on visualizations** - Get feedback from teams
3. **Expand to all repositories** - Roll out automated collection
4. **Share success stories** - Demonstrate ROI and adoption

---

## Resources

- [BigQuery Documentation](https://cloud.google.com/bigquery/docs)
- [Grafana Dashboard Best Practices](https://grafana.com/docs/grafana/latest/dashboards/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Data Structure Specification](data-structure.md)
- [Tracking Guide](tracking-guide.md)

---

**Note**: This is a living document. Integration details will evolve as we pilot and refine the implementation.
