---
title: "Jinja Basics: Getting Started"
description: "Introduction to Jinja templating language and its basic syntax."
linkTitle: "Jinja Basics"
weight: 10
---

## Introduction
Jinja is a modern and designer-friendly templating language for Python. In FortiSOAR, it's used for dynamic content generation, data transformation, and automation tasks.

## Basic Syntax

### Variables
Variables in Jinja are enclosed in double curly braces:
```jinja
{{ variable_name }}
{{ user.name }}
{{ dict['key'] }}
```

### Expressions
You can use expressions inside variable blocks:
```jinja
{{ number + 1 }}
{{ string.upper() }}
{{ list|length }}
```

### Comments
```jinja
{# This is a comment #}
```

## Hands-On Exercise 1: Basic Variable Usage

### Task
Given the following data structure:
```python
user = {
    'name': 'John Doe',
    'age': 30,
    'role': 'analyst',
    'permissions': ['read', 'write']
}
```

Create Jinja templates to:
1. Display the user's name
2. Show their role in uppercase
3. Show their age plus 5 years
4. Check if they have 'write' permission

### Solution
{{% expand title="Click to see the Solution" %}}
```jinja
Name: {{ user.name }}
Role: {{ user.role | upper }}
Age in 5 years: {{ user.age + 5 }}
Has write permission: {{ 'write' in user.permissions }}
```
{{% /expand %}}
## Control Structures

### If Statements
```jinja
{% if condition %}
    Content if true
{% elif other_condition %}
    Content if other condition is true
{% else %}
    Content if all conditions are false
{% endif %}
```

### For Loops
```jinja
{% for item in list %}
    {{ item }}
{% endfor %}

{# With index #}
{% for item in list %}
    {{ loop.index }}: {{ item }}
{% endfor %}
```

## Hands-On Exercise 2: Control Flow

### Task
Given this alerts list:
```python
alerts = [
    {'severity': 'high', 'status': 'open', 'title': 'Server Down'},
    {'severity': 'medium', 'status': 'closed', 'title': 'High CPU Usage'},
    {'severity': 'high', 'status': 'open', 'title': 'Malware Detected'},
    {'severity': 'low', 'status': 'open', 'title': 'Update Available'}
]
```

Create a template that:
1. Lists only open alerts
2. Shows alert number and title
3. Adds an "URGENT!" prefix for high severity alerts
4. Shows a total count at the end

### Solution
{{% expand title="Click to see the Solution" %}}
```jinja
Open Alerts:
{% set open_count = 0 %}
{% for alert in alerts %}
    {% if alert.status == 'open' %}
        {% set open_count = open_count + 1 %}
        {{ loop.index }}. {% if alert.severity == 'high' %}URGENT! {% endif %}{{ alert.title }}
    {% endif %}
{% endfor %}

Total open alerts: {{ open_count }}
```
{{% /expand %}}

## Challenge: Alert Summary Generator

### Task
Create a template that generates an alert summary with the following requirements:

1. Group alerts by severity
2. Show count for each severity
3. Only include open alerts
4. Add custom icons:
   - High: 🔴
   - Medium: 🟡
   - Low: 🟢
5. Sort alert titles alphabetically within each severity group

### Starting Data
```python
alerts = [
    {'severity': 'high', 'status': 'open', 'title': 'Data Breach'},
    {'severity': 'high', 'status': 'closed', 'title': 'Server Crash'},
    {'severity': 'medium', 'status': 'open', 'title': 'Failed Login'},
    {'severity': 'low', 'status': 'open', 'title': 'Certificate Expiring'},
    {'severity': 'medium', 'status': 'open', 'title': 'Disk Space Low'},
    {'severity': 'high', 'status': 'open', 'title': 'Malware Detected'},
    {'severity': 'low', 'status': 'open', 'title': 'Updates Available'}
]
```

Try to solve this on your own before checking the solution!

### Hints
1. Use nested for loops
2. Consider using a dictionary to group alerts
3. Remember to use the `|length` filter for counting
4. The `|sort` filter can help with sorting

{{% expand title="Click to see the Solution" %}}

```jinja
{# First, group alerts by severity #}
{% set severity_groups = {'high': [], 'medium': [], 'low': []} %}
{% for alert in alerts %}
    {% if alert.status == 'open' %}
        {% if alert.severity == 'high' %}
            {% do severity_groups.high.append(alert) %}
        {% elif alert.severity == 'medium' %}
            {% do severity_groups.medium.append(alert) %}
        {% else %}
            {% do severity_groups.low.append(alert) %}
        {% endif %}
    {% endif %}
{% endfor %}

Alert Summary Report
==================

{% for severity, alerts in severity_groups.items() %}
{% set icon = '🔴' if severity == 'high' else '🟡' if severity == 'medium' else '🟢' %}
{{ severity|upper }} Alerts ({{ alerts|length }}) {{ icon }}
------------------
{% for alert in alerts|sort(attribute='title') %}
  - {{ alert.title }}
{% endfor %}

{% endfor %}
```
{{% /expand %}}


## Next Steps
In the next section, we'll explore FortiSOAR's custom filters and more advanced Jinja features.