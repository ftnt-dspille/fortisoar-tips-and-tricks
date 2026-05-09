---
title: FortiSOAR Custom Filters and Functions
linkTitle: Custom Filters and Functions
weight: 20
---


This section covers FortiSOAR's custom filters and functions, with practical examples and exercises.

## Date and Time Operations

### Available Functions

- `get_current_date()`: Returns current date
- `get_current_datetime()`: Returns current date and time
- `currentdateminus(days)`: Returns timestamp minus specified days
- Arrow library for advanced date operations

### Basic Usage

```jinja
Current Date: {{ get_current_date() }}
Yesterday: {{ currentdateminus(1) }}
Formatted DateTime: {{ arrow.utcnow().format('YYYY-MM-DD HH:mm:ss') }}
```

## Hands-On Exercise 1: Date Formatting

### Task

Create a template that:

1. Shows current time in three different timezones (UTC, EST, PST)
2. Calculates a date range for the last 7 days
3. Formats dates in a human-readable format

### Solution

{{% expand title="Click to see the Solution" %}}

```jinja
{% set now = arrow.utcnow() %}
{% set week_ago = arrow.utcnow().shift(days=-7) %}

Current Times:
UTC: {{ now.format('YYYY-MM-DD HH:mm:ss') }}
EST: {{ now.to('US/Eastern').format('YYYY-MM-DD HH:mm:ss') }}
PST: {{ now.to('US/Pacific').format('YYYY-MM-DD HH:mm:ss') }}

Date Range:
Start: {{ week_ago.format('MMMM D, YYYY') }}
End: {{ now.format('MMMM D, YYYY') }}
```

{{% /expand %}}

## FortiSOAR Data Filters

### IRI Operations

```jinja
{# Basic IRI resolution #}
{{ '/api/3/alerts/123' | fromIRI }}

{# With relationship data #}
{{ ('/api/3/alerts/123?$relationships=true' | fromIRI).indicators }}
```

### Data Transformation

```jinja
{# Convert to JSON #}
{{ data | toJSON }}

{# Create HTML table #}
{{ data | json2html(['field1', 'field2']) }}

{# Count occurrences #}
{{ list | count_occurrence }}
```

## Hands-On Exercise 2: Data Transformation

### Task

Given the following data:

```python
items = [
    {'type': 'alert', 'severity': 'high', 'source': 'firewall'},
    {'type': 'alert', 'severity': 'medium', 'source': 'ids'},
    {'type': 'incident', 'severity': 'high', 'source': 'siem'},
    {'type': 'alert', 'severity': 'low', 'source': 'firewall'}
]
```

Create templates to:

1. Count items by type and severity
2. Generate an HTML table of high severity items
3. Create a JSON summary with counts

### Solution

{{% expand title="Click to see the Solution" %}}

```jinja
{# Count by type #}
{% set type_counts = items | map(attribute='type') | list | count_occurrence %}
Types: {{ type_counts | toJSON }}

{# High severity items table #}
{{ items | selectattr('severity', 'equalto', 'high') | list | json2html(['type', 'source']) }}

{# JSON summary #}
{% set summary = {
    'counts': type_counts,
    'high_severity_count': items | selectattr('severity', 'equalto', 'high') | list | length,
    'sources': items | map(attribute='source') | unique | list
} %}
{{ summary | toJSON }}
```

{{% /expand %}}

## Picklist and Relationship Operations

### Picklist Usage

```jinja
{# Get picklist item #}
{{ "AlertStatus" | picklist("Open") }}

{# Get specific field #}
{{ "AlertSeverity" | picklist("High", "@id") }}
```

### Relationship Loading

```jinja
{# Load all relationships #}
{{ alert_iri | loadRelationships('indicators') }}

{# Load specific fields #}
{{ incident_iri | loadRelationships('alerts', ['title', 'severity']) }}
```

## Challenge: Alert Processing System

### Task

Create a comprehensive alert processing template that:

1. Groups alerts by severity using picklists
2. Loads related indicators for each alert
3. Generates an HTML report with:
    - Alert counts by severity
    - Related indicator counts
    - Timeline of alerts
4. Includes timezone-aware timestamps
5. Exports summary data as JSON

### Starting Data

```python
alert_iris = [
    '/api/3/alerts/123',
    '/api/3/alerts/124',
    '/api/3/alerts/125'
]
```

### Hints

1. Use `fromIRI` to load alert details
2. Use `loadRelationships` for indicators
3. Use arrow for timestamp formatting
4. Consider using `json2html` for report sections

{{% expand title="Expand me..." %}}

```jinja
{# Load alert data #}
{% set alerts = [] %}
{% for iri in alert_iris %}
    {% set alert = iri | fromIRI %}
    {% set indicators = iri | loadRelationships('indicators') %}
    {% do alerts.append({
        'details': alert,
        'indicators': indicators,
        'severity': alert.severity | picklist(alert.severity.itemValue),
        'timestamp': arrow.get(alert.createDate)
    }) %}
{% endfor %}

{# Generate report #}
Alert Processing Report
=====================

Severity Summary:
{{ alerts | groupby('severity.itemValue') | map(attribute='list') | map('length') | json2html }}

Timeline:
{% for alert in alerts | sort(attribute='timestamp') %}
{{ alert.timestamp.format('YYYY-MM-DD HH:mm:ss') }} - {{ alert.details.title }}
    Indicators: {{ alert.indicators | length }}
{% endfor %}

{# Export JSON summary #}
{% set summary = {
    'total_alerts': alerts | length,
    'total_indicators': alerts | sum(attribute='indicators') | length,
    'severity_distribution': alerts | groupby('severity.itemValue') | map(attribute='list') | map('length') | list
} %}
{{ summary | toJSON }}
```

{{% /expand %}}

## Next Steps

In the next section, we'll explore advanced Jinja features and complex templating patterns.