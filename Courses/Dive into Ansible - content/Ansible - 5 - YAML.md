- Ansible Playbooks utilise YAML, as human reaedable, data-serialisation language
- https://yaml.org/
- https://en.wikipedia.org/wiki/YAML

```yaml
# Every YAML file should start with three dashes (optional)
---
example_key_1: this is a string
example_key_2: this is another string
#Every YAML file should end with three dots (optional)
...
```

```yaml
---
no_quotes: this is a string example
double_quotes: "this is a string example"
single_quotes: 'this is a string example'
...
```

```yaml
---
example_key_1: |
  this is a string
  that goes over
  multiple lines
...
```

```yaml
---
# or example_key_1: >- to remove last character(\n)
example_key_1: >
  this is a string
  that goes over
  multiple lines
...
```

```yaml
---
example_integer: 1
...
```

```yaml
---
{example_key_1: example_value_1, example_key_2: example_value_2}
...
```

```yaml
---
[example_list_entry1, example_list_entry2]
...
```

```yaml
---
example_key_1:
  sub_example_key1: sub_example_value1

example_key_2:
  sub_example_key2: sub_example_value2
...
```

```yaml
---
example_1:
- item_1
- item_2
- item_3

example_2:
- item_4
- item_5
- item_6
...
```

```yaml
---
example_dictionary_1:
  - example_1:
    - item_1
    - item_2
    - item_3
  - example_2:
    - item_4
    - item_5
    - item_6
...
```

```yaml
---
Aston Martin:
  year_founded: 1913
  website: astonmartin.com
  founded_by:
    - Lionel Martin
    - Robert Bamford

Fiat:
  year_founded: 1899
  website: fiat.com
  founded_by:
    - Giovanni Agnelli

Ford:
  year_founded: 1903
  website: ford.com
  founded_by: 
    - Henry Ford

Vauxhall:
  year_founded: 1857
  website: vauxhall.co.uk
  founded_by:
    - Alexander Wilson
...
```

