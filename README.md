# dotstar

**Wildcard patterns for dotget paths.**

```python
>>> from dotstar import search
>>> data = {"users": [{"name": "Alice", "email": "alice@example.com"}, 
...                   {"name": "Bob", "email": "bob@example.com"}]}
>>> search(data, "users.*.email")
['alice@example.com', 'bob@example.com']
```

## Why?

[dotget](https://github.com/yourusername/dotget) gives you simple paths to single values.

Sometimes you need all matching values. That's what dotstar does.

## Install

```bash
pip install dotstar
```

Requires Python 3.6+. No other dependencies.

## The Star of the Show

One new symbol: `*` means "all keys" or "all indices".

```python
from dotstar import search

data = {
    "departments": {
        "engineering": {
            "employees": [
                {"name": "Alice", "level": "senior"},
                {"name": "Bob", "level": "junior"}
            ]
        },
        "sales": {
            "employees": [
                {"name": "Carol", "level": "senior"},
                {"name": "Dan", "level": "junior"}
            ]
        }
    }
}

# Get all employee names across all departments
names = search(data, "departments.*.employees.*.name")
# ['Alice', 'Bob', 'Carol', 'Dan']

# Get all senior employees
seniors = search(data, "departments.*.employees.*.level")
# ['senior', 'junior', 'senior', 'junior']
```

## Finding Paths

Sometimes you need to know where values came from:

```python
from dotstar import find_all

# Find all paths and values
results = find_all(data, "departments.*.employees.*.name")
# [
#   ('departments.engineering.employees.0.name', 'Alice'),
#   ('departments.engineering.employees.1.name', 'Bob'),
#   ('departments.sales.employees.0.name', 'Carol'),
#   ('departments.sales.employees.1.name', 'Dan')
# ]

# Now you know exactly where each value lives
for path, name in results:
    print(f"{name} is at {path}")
```

## Pattern Objects

For reusable patterns:

```python
from dotstar import Pattern

# Define reusable patterns
ALL_USERS = Pattern("users.*")
ALL_EMAILS = ALL_USERS / "email"
ACTIVE_USERS = Pattern("users.*.status")

# Use them on different data
emails = ALL_EMAILS.search(data1)
more_emails = ALL_EMAILS.search(data2)
```

## Works with dotget

dotstar is designed to complement dotget:

```python
from dotget import get
from dotstar import search

# Get one user's email
email = get(data, "users.0.email")

# Get all users' emails  
emails = search(data, "users.*.email")
```

Same syntax, different results. Use the right tool for the job.

## When to use dotstar

✅ You need all matching values from nested data  
✅ You're extracting data for analysis or transformation  
✅ You want to find patterns in JSON responses  
✅ You're building a simple data query tool  

## When NOT to use dotstar

❌ You need complex queries (use JMESPath or JSONPath)  
❌ You need conditional filtering (use list comprehensions)  
❌ You're working with huge datasets (this loads everything)  
❌ You need aggregations or transformations (use pandas)  

## The Pattern Language

- **Dots** traverse objects: `user.name`
- **Numbers** index arrays: `items.0`
- **Stars** match all: `users.*`

You can combine them:
- `users.*.addresses.0.city` - First address city for all users
- `data.*.*.id` - All IDs two levels deep
- `response.items.*` - All items in response

## Philosophy

dotstar does one thing: find all values matching a pattern. It's not trying to be:
- A query language (that's JMESPath)
- A transformation tool (that's jq)  
- A validation library (that's jsonschema)

It's just wildcards for paths. Simple enough to understand immediately, powerful enough to be useful.

## Performance Note

dotstar is designed for convenience, not speed. It:
- Traverses the entire data structure
- Returns all matches in a list
- Works best with reasonably-sized data

For large datasets or performance-critical code, consider streaming approaches or specialized tools.

## License

MIT. Use it however you like.

---

*"Make it as simple as possible, but not simpler."* - Albert Einstein
