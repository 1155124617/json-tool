---
name: "json-tool"
description: "Lightweight JSON reader/editor CLI for agents. Read, edit, delete, or add JSON nodes by path without loading the whole file into context."
---

# json-tool

A lightweight JSON reader/editor CLI designed for AI agents. Instead of reading an entire JSON file into context and doing text replacement, use path-based operations to target specific nodes.

## Installation

Place `json-tool` on your PATH:

```bash
chmod +x ~/.openclaw/workspace/scripts/json-tool
# Add to PATH or symlink
ln -s ~/.openclaw/workspace/scripts/json-tool ~/.local/bin/json-tool
```

Requires Python 3.8+.

## Usage

### read — Read a JSON node

```bash
json-tool read <file> <path>
```

**Examples:**

```bash
# Read a nested object field
json-tool read config.json "database.host"
# → "localhost"

# Read an array element
json-tool read users.json "users[0].name"
# → "Alice"

# Read the last array element
json-tool read users.json "users[-1].email"
# → "bob@example.com"

# Read entire array
json-tool read data.json "items"
# → [{"id": 1}, {"id": 2}]

# Read root (print whole file compactly)
json-tool read data.json "."
```

### edit — Edit a JSON node

```bash
json-tool edit <file> <path> <value>
```

**Value parsing (auto-detected):**

| Input | Parsed as |
|-------|-----------|
| `42` | number |
| `3.14` | number (float) |
| `true` / `false` | boolean |
| `null` | null |
| `"hello"` | string (with quotes) |
| `{"a":1}` | JSON object |
| `[1,2,3]` | JSON array |
| `anything_else` | string literal |

**Examples:**

```bash
# Update a simple field
json-tool edit config.json "debug" true

# Update a nested field
json-tool edit config.json "database.port" 5432

# Update with a JSON object
json-tool edit config.json "database.options" '{"ssl": true, "timeout": 30}'

# Update array element
json-tool edit data.json "users[0].role" "\"admin\""
```

### delete — Delete a JSON node

```bash
json-tool delete <file> <path>
```

**Examples:**

```bash
# Delete a key from object
json-tool delete config.json "deprecated_key"

# Delete an array element
json-tool delete data.json "users[2]"
```

### add — Add/insert a JSON node (creates intermediates)

```bash
json-tool add <file> <path> <value>
```

**Examples:**

```bash
# Add a new field (auto-creates nested dicts if needed)
json-tool add config.json "new_section.feature" "enabled"

# Append to array
json-tool add data.json "users[3]" '{"name": "Charlie"}'

# Create a new file with structure
json-tool add new-config.json "app.name" "MyApp"
```

### type — Show the type of a node

```bash
json-tool type <file> <path>
```

**Examples:**

```bash
json-tool type config.json "debug"
# → boolean

json-tool type data.json "users"
# → list
```

## Path Syntax

Paths use dot notation with bracket array indexing:

```
users[0].name          # First user's name
config.database.host   # Nested object
data.items[-1]         # Last array element
users[*]               # All users (read only)
```

## Aliases

| Primary | Aliases |
|---------|---------|
| read | get |
| edit | set |
| delete | del, rm |

## Agent Workflow Example

**Before (text-based editing):**
```
1. Read entire 5MB config.json into context
2. Find the line with "database.port": 5432
3. Use edit tool to replace text
4. Risk: might corrupt JSON structure
```

**After (path-based editing):**
```
1. json-tool read config.json "database.host"   # Check current value
2. json-tool edit config.json "database.port" 3306  # Update only this node
3. json-tool read config.json "database"        # Verify the change
```

## Design Rationale

1. **Path-based targeting** — Agents specify *what* to change, not *where* in the text
2. **Type-aware values** — Numbers, booleans, null, objects, arrays are auto-detected
3. **Auto-create intermediates** — `add` command creates missing parent dicts/lists
4. **Preserves formatting** — File is re-serialized with standard JSON formatting
5. **Safe operations** — Invalid paths return clear error messages, never silently corrupt

## Limitations

- Currently loads the entire file into memory (not truly streaming)
- For very large files (>100MB), consider splitting into smaller files or using SQLite
- Does not preserve custom formatting (comments, key order, spacing)
- Values with spaces should be quoted: `'"hello world"'`

## Integration with OpenClaw

Add to your agent's tool set:

```bash
# In agent workspace or as a skill
ln -s ~/.openclaw/workspace/scripts/json-tool ~/.local/bin/json-tool
```

Then the agent can use it in reasoning:
> "I'll check the current value first: `json-tool read config.json 'debug'`"
> "Then update it: `json-tool edit config.json 'debug' true`"
