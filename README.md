# `dotstar`: Simple Wildcard Addressing

> "Things should be made as simple as possible, but not simpler."
>
> — (Probably not) Albert Einstein

**Find all values in a nested data structure that match a simple wildcard pattern.**

```python
>>> from dotstar import search
>>> data = {"users": [{"name": "Alice"}, {"name": "Bob"}]}
>>> search(data, "users.*.name")
['Alice', 'Bob']
```

## The Philosophy: Simple Patterns

`dotstar` is the second layer in the `dot` ecosystem's **Addressing Pillar**. It introduces a single, powerful concept: the wildcard (`*`).

1. **`dotget` (Exact Addressing):** For when you know the *exact* path to a value (`users.0.name`).
2. **`dotstar` (Pattern Addressing):** For when you need to gather all items that match a simple wildcard pattern (`users.*.name`). It is the simplest tool for structural matching.
3. **`dotselect` (Advanced Selection):** For when you need more complex queries, like attribute checks (`[name=Alice]`), deep searches (`..name`), and custom logic.

`dotstar` is for when you know the *shape* of the data you're looking for, but not the exact keys or indices. It's for gathering all children under a common parent, or all values for a specific key across a list of objects.

## Install

```bash
pip install dotstar
```

## Usage

### As a Library

`dotstar` provides two main functions: `search` (to get values) and `find_all` (to get `(path, value)` tuples).

```python
from dotstar import search, find_all

data = {
    "users": [
        {"id": 1, "name": "Alice", "status": "active"},
        {"id": 2, "name": "Bob", "status": "inactive"},
    ],
    "groups": {
        "admin": {"members": [1]},
        "user": {"members": [2]}
    }
}

# Get all user names
names = search(data, "users.*.name")
# -> ['Alice', 'Bob']

# Get all member IDs from all groups
member_ids = search(data, "groups.*.members.*")
# -> [1, 2]

# Find the exact paths of all user statuses
statuses = find_all(data, "users.*.status")
# -> [('users.0.status', 'active'), ('users.1.status', 'inactive')]
```

The `find_all` function is useful for discovering paths that you can then use with other tools, like `dotmod` or `dotbatch`.

### From the Command Line

`dotstar` reads JSON from stdin and prints matching values to stdout.

```sh
# Find all user names in a JSON file
$ cat users.json | dotstar "users.*.name"
"Alice"
"Bob"
```

## Boundaries: When to Use `dotstar`

Use `dotstar` when you need to:
✅ Get all items from a list.
✅ Get all values from a dictionary.
✅ Extract all values for a specific key from a list of objects (e.g., all `name`s from all `users`).
✅ Find the exact paths of multiple items that match a structural pattern.

Do **not** use `dotstar` when you need to:
❌ Get just one specific value. **Use `dotget`**.
❌ Filter items based on their values (`[status=active]`). **Use `dotselect`**.
❌ Perform deep, recursive searches for a key (`..name`). **Use `dotselect`**.
❌ Modify data in place. **Use `dotmod`**.

## "Steal This Code"

Don't want a dependency? The core of `dotstar` is a single recursive function. Copy it.

```python
def search(data, pattern):
    """Searches data for a dot-separated pattern with '*' wildcards."""
    results = []
    
    def _search_recursive(sub_data, segments):
        if not segments:
            results.append(sub_data)
            return

        segment = segments[0]
        remaining = segments[1:]

        if segment == '*':
            if isinstance(sub_data, dict):
                for value in sub_data.values():
                    _search_recursive(value, remaining)
            elif isinstance(sub_data, list):
                for item in sub_data:
                    _search_recursive(item, remaining)
        else:
            try:
                if isinstance(sub_data, list) and segment.isdigit():
                    next_data = sub_data[int(segment)]
                elif isinstance(sub_data, dict):
                    next_data = sub_data[segment]
                else:
                    return
                _search_recursive(next_data, remaining)
            except (KeyError, IndexError, TypeError):
                return

    _search_recursive(data, pattern.split('.'))
    return results
```
