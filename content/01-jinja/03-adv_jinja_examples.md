---
title: "Advanced Usage Examples"

weight: 30
---

These examples are things I have used in the past and frequently refer to. They are not exhaustive, but they are a good starting point for more complex Jinja2 usage.

## Regular Expressions

### IP Address Pattern Matching

```jinja
{{ mal | map('regex_replace', 
    '^([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})\.([0-9]{1,3})$', 
    '\\1[.]\\2.\\3.\\4') | list }}
```

### Filter Phone Numbers

```jinja
{% for indicator in iocs %}
    {% if not indicator.value | regex_search('\+[0-9]{11}') %}
        {{indicator.value}}
    {% endif %}
{% endfor %}
```

## Data Transformation

### JSON to HTML Table

```jinja
{{ vars.your_list_of_dictionaries | json2html([list, of, attributes, you, want, in, table]) }}

# With styling options
{{ data | json2html(
    row_fields=["row_1"],
    template="Stylized",
    display="Horizontal",
    styling=true,
    table_style={'table class=\'cs-data-table\'': 'max-width: 500px;', 'td': 'white-space: break-spaces;'}
) }}
```

### Text to Dictionary Conversion

```jinja
{% set myDict = {} %}
{% for line in text.split("\r\n") %}
    {% set key,value = line.split(":",1) %}
    {{ "" if myDict.update({key:value}) }}
{% endfor %}
{{myDict}}
```

## List Operations

### Reverse a List

```jinja
{% set reverse_iter = vars.steps.Reiterate_Pages.total_ids | reverse %}
{% set mylist = [] %}
{% for item in reverse_iter %}
    {{ "" if mylist.append(item) }}
{% endfor %}
{{mylist}}
```

## Date and Time Operations

### Arrow Date Examples

```jinja
# Current UTC time
{{arrow.utcnow().format('YYYY-MM-DD HH:mm:ss')}}

# Time with timezone shift
{{arrow.get(tzinfo="US/Central").shift(hours=-2).format('YYYY-MM-DDTHH:mm:ss[Z]')}}

# Get this Monday
{{ arrow.get().shift(days=(-((arrow.get().format("d") | int)-1))).format("M/D") }}

# Get next Sunday
{{ arrow.get().shift(days=(7-((arrow.get().format("d") | int)))).format("M/D") }}
```

## YAQL Filter Examples

### Basic YAQL Operations

```jinja
# Get specific value
{{ {"var1":1,"var2":"a"} | yaql('$.var1') }}

# String manipulation
{{ "test" | yaql('$.toUpper()') }}

# Filter non-False data
{{ data | yaql('dict($.items().where(bool($[1])))') }}
```

## User and Identity Operations

### Set Assigned To Field by Email

```jinja
{% set ns = namespace(iri ="") %}
{% for user in (("/api/3/people" | fromIRI)['hydra:member']) %}
    {% if user.email == vars.input.params['api_body'].alert.username %}
        {% set ns.iri = user['@id'] %}
    {% endif %}
{% endfor %}
{{ns.iri | default("/api/3/people/default-id", true)}}
```

### Set Assigned To Field by Name

```jinja
{% set ns = namespace(iri ="") %}
{% set firstname = vars.sourcedata.assigned_to.split(' ')[0] %}
{% set lastname = vars.sourcedata.assigned_to.split(' ')[1] %}
{% for user in (("/api/3/people?firstname=" + firstname + "&lastname=" + lastname) | fromIRI)['hydra:member'] %}
    {% set ns.iri = user['@id'] if user else None %}
{% endfor %}
{{ns.iri}}
```

## IP and Network Operations

### Filter IPs by CIDR Range

```jinja
{% set ignored_ips = [] %}
{% set wanted_ips = [] %}
{% for ip_addr in vars.ip_iocs %}
    {% set match_results = [] %}
    {% for cidr_range in vars.excluded_Cidrs %}
        {% set match_result = ip_addr | ip_range(cidr_range) %}
        {% do match_results.append(match_result) %}
    {% endfor %}
    {% if match_results | json_query('[?ip_matched]') | length > 0 %}
        {% do ignored_ips.append(ip_addr) %}
    {% else %}
        {% do wanted_ips.append(ip_addr) %}
    {% endif %}
{% endfor %}
```

## CSV Export

### Export Records as CSV with Formatting

```jinja
{% for item in vars.steps.Run_Query.data['hydra:member'] %}
    {% for k,v in item.items() %}
        {% if v is mapping and '/api/3/picklists' in v.get('@id', 'N/A') %}
            {{item.update({k: v.itemValue })}}
        {% elif v is mapping and '/api/3/people/' in v.get('@id', 'N/A') %}
            {% set fn = v.firstname %}
            {% set ln = v.lastname %}
            {{item.update({k:fn+' '+ln})}}
        {% elif k in vars.dateFields %}
            {{item.update({k:(arrow.get((v | int | abs)).to(vars.timezone).format(vars.dateFormat) if v else '')})}}
        {% endif %}
    {% endfor %}
    {{vars.record_list.append(item)}}
{% endfor %}
```