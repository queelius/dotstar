# `dotstar`

**Simple wildcard addressing for nested data structures.**

```python
>>> from dotstar import search
>>> data = {"users": [{"name": "Alice"}, {"name": "Bob"}]}
>>> search(data, "users.*.name")
['Alice', 'Bob']
```

## Why? Pattern-Based Addressing

The `dot` ecosystem provides a layered approach to finding data:

* `dotget` uses **exact addressing** (`users.0`).
* `dotstar` introduces **pattern addressing** (`users.*`).
* `dotquery` provides **conditional addressing** (`users[status=active]`).

`dotstar` is the simplest way to get all values that match a structural pattern, without needing to know the specific keys or indices.

## Install

```bash
pip install dotstar
```

## Usage

`dotstar` can be used as a Python library or as a command-line tool.

### CLI

`dotstar` reads JSON or YAML from stdin and prints the results to stdout.

```bash
# Find all values matching the pattern
cat data.json | dotstar "users.*.name"

# Specify YAML input
cat data.yaml | dotstar --input-format yaml "users.*.name"

# Find all paths and values
cat data.json | dotstar --find "users.*.name"
```

### Library

```python
from dotstar import search, find_all

data = {"users": [{"name": "Alice"}, {"name": "Bob"}]}

# Get all values matching a pattern
search(data, "users.*.name")
# -> ['Alice', 'Bob']

# Get all (path, value) tuples matching a pattern
find_all(data, "users.*.name")
# -> [('users.0.name', 'Alice'), ('users.1.name', 'Bob')]
```

## The `*` Wildcard

The star (`*`) is the one piece of syntax `dotstar` adds. It means "match all items in a list" or "match all values in a dictionary."

```python
from dotstar import search

data = {"sections": {"a": {"val": 1}, "b": {"val": 2}}}

# Get all 'val' keys from all sections
values = search(data, "sections.*.val")
# -> [1, 2]
```

## Finding Paths for Composition

Sometimes you need to know *where* the values came from. This is crucial for composing with other tools, like `dotbatch`. The `find_all` function returns both the exact path and the value.

```python
from dotstar import find_all

data = {"users": [{"name": "Alice"}, {"name": "Bob"}]}
results = find_all(data, "users.*.name")
# -> [('users.0.name', 'Alice'), ('users.1.name', 'Bob')]

# This output can now be used to build a dotbatch transaction
# to modify Alice's name without knowing her index beforehand.
```

## When to use `dotstar`

✅ You need all values that match a simple structural pattern.
✅ You're extracting data for analysis or transformation.
✅ You need to find the exact paths of multiple items to feed into another tool.

## When NOT to use `dotstar`

❌ You need conditional filtering (e.g., `[key=value]`). **Use `dotquery`**. `dotstar` deliberately omits these for simplicity.
❌ You only need a single, known path. **Use `dotget`**.

## Philosophy: The Butter Knife

`dotstar` does one thing: find all paths matching a wildcard pattern. It is the simple, safe, and reliable "butter knife" of the addressing layer.

It is not a query language. It is not a transformation tool. It is the pure and simple tool for when you need a wildcard, and nothing more. This simplicity is a feature.

## License

MIT. Use it however you like.

---

> "Make it as simple as possible, but not simpler." - Albert Einstein
