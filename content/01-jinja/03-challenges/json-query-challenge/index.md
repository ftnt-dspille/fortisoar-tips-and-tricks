---
title: "JSON Query Challenge"
description: "Test your Jinja skills with JSON querying and filtering"
weight: 20
---

## Introduction

This challenge will test your Jinja abilities using JSON query filters. If you get stuck, refer to the comprehensive list of Jinja filters in the [FortiSOAR documentation](https://docs.fortinet.com/document/fortisoar/latest/playbooks-guide/767891/jinja-filters-and-functions#Comprehensive_list_of_filters).

For a primer on JSON Query, check out the [JMESPath tutorial](https://jmespath.org/tutorial.html).

## Getting Started

{{% notice style="primary" title="Setup Instructions" %}}

1. Download the sample Pokemon data file
2. Use the FortiSOAR Jinja Editor for the best experience (it includes all required filters)
3. Download the Challenge JSON file
3. Paste the JSON content into the editor
   {{% /notice %}}

### Challenge Data

{{% resources title="Required Files" style="info" pattern=".*\.json" /%}}

Your editor should look like this:
![Jinja Editor Setup](images/json_query_challenge.png)

## Challenges

### Section 1: JSON Query Basics

All challenges in this section use the `json_query` filter, which implements JMESPath queries.

#### Challenge 1: Basic Querying

Display all of the English names

{{% expand "Solution 1" %}}

```jinja
{{ data.pokemon | json_query('[].name.english') }}
```

{{% /expand %}}

#### Challenge 2: Conditional Filtering

Display the original Pokemon dictionaries where the IDs are less than 50

{{% expand "Solution 2" %}}

```jinja
{{ data.pokemon | json_query('[?id <= `50`]') }}
```

**Bonus**: Display French names of those same 50 Pokemon

```jinja
{{ data.pokemon | json_query('[?id <= `50`].name.french') }}
```

{{% /expand %}}

#### Challenge 3: Complex Filtering

Find the strongest Pokemon with an Attack value over 150

{{% expand "Solution 3" %}}

```jinja
{{ data.pokemon | json_query('[?base.Attack > `150`]') }}
```

**Bonus**: Create a list with just names and Attack values

```jinja
{{ data.pokemon | json_query('[?base.Attack > `150`].[name.english, base.Attack]') }}
```

{{% /expand %}}

#### Challenge 4: Data Transformation

Find Pokemon with HP under 25 and display their names

{{% expand "Solution 4" %}}

```jinja
{{ data.pokemon | json_query('[?base.HP < `25`].name.english') }}
```

**Bonus**: Create a dictionary with names and HP

```jinja
{{ data.pokemon | json_query('[?base.HP < `25`].{name: name.english, HP: base.HP}') }}
```

{{% /expand %}}

#### Challenge 5: Sorting

Create an alphabetically sorted list of Pokemon names

{{% expand "Solution 5" %}}

```jinja
{{ data.pokemon | json_query('[].name.english | sort(@)') }}
```

**Bonus**: Reverse the sort

```jinja
{{ data.pokemon | json_query('[].name.english | sort(@) | reverse(@)') }}
```

{{% /expand %}}

#### Challenge 6: Aggregation

Find the Pokemon with the highest HP

{{% expand "Solution 6" %}}

```jinja
{{ data | json_query('max_by(pokemon,&base.HP).{name:name.english, HP:base.HP}') }}
```

**Bonus**: Find the lowest HP

```jinja
{{ data | json_query('min_by(pokemon,&base.HP).{name:name.english, HP:base.HP}') }}
```

{{% /expand %}}

#### Challenge 7: Summation

Calculate the total HP of all Pokemon

{{% expand "Solution 7" %}}

```jinja
{{ data.pokemon | json_query('[].base.HP | sum(@)') }}
```

{{% /expand %}}

#### Challenge 8: String Operations

Find Pokemon names starting with "Char"

{{% expand "Solution 8" %}}

```jinja
{{ data.pokemon | json_query('[?starts_with(@.name.english, `Char`)].name.english') }}
```

**Bonus**: Find names ending with 'd'

```jinja
{{ data.pokemon | json_query('[?ends_with(@.name.english, `d`)].name.english') }}
```

{{% /expand %}}

#### Challenge 9: Type Filtering

Find all Bug-type Pokemon

{{% expand "Solution 9" %}}

```jinja
{{ data.pokemon | json_query('[?contains(@.type, `Bug`)]') }}
```

**Bonus 1**: Find Bug types that aren't Poison

```jinja
{{ data.pokemon | json_query('[?contains(@.type, `Bug`) && !contains(@.type, `Poison`)]') }}
```

**Bonus 2**: Find Fire types

```jinja
{{ data.pokemon | json_query('[?contains(@.type, `Fire`)]') }}
```

{{% /expand %}}

#### Challenge 10: Complex Type Filtering

##### Part 1

Find Water-type Pokemon with HP above 100

{{% expand "Solution 10" %}}

```jinja
{{ data.pokemon | json_query('[?base.HP > `100` && contains(@.type, `Water`)]') }}
```

{{% /expand %}}

##### Part 2

**Bonus**: Find Fire types with HP below 50 and Attack above 58
{{% expand "Bonus" %}}

```jinja
{{ data.pokemon | json_query('[?base.HP < `50` && base.Attack > `58` && contains(@.type, `Fire`)]') }}
```

{{% /expand %}}

#### Challenge 11: Multiple Conditions

Find all Water or Fire type Pokemon

{{% expand "Solution 11" %}}

```jinja
{{ data.pokemon | json_query('[?contains(@.type, `Water`) || contains(@.type, `Fire`)]') }}
```

{{% /expand %}}

#### Challenge 12: Unique Values

Display a unique list of all Pokemon types

{{% expand "Solution 12" %}}

```jinja
{{ data.pokemon | json_query('[].type[]') | unique }}
```

{{% /expand %}}

## Next Steps

{{% notice style="tip" title="Further Practice" %}}

1. Try combining multiple filters
2. Create your own complex queries
3. Experiment with different data transformations
   {{% /notice %}}