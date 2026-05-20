# json-tool

A lightweight, zero-dependency JSON reader/editor CLI designed for AI agents.

Instead of loading entire JSON files into context and doing fragile text replacement, use **path-based operations** to target specific nodes directly.

---

## ✨ Why?

| Before (text-based) | After (path-based) |
|---|---|
| Read entire 5MB config.json into context | `json-tool read config.json "database.host"` |
| Find line with `"port": 5432` | `json-tool edit config.json "database.port" 3306` |
| Risk corrupting JSON structure | Type-safe, structure-preserving edits |

---

## 🚀 Installation

### Option 1: Direct download

```bash
curl -O https://raw.githubusercontent.com/1155124617/json-tool/master/json-tool
chmod +x json-tool
```

### Option 2: Clone repository

```bash
git clone https://github.com/1155124617/json-tool.git
cd json-tool
chmod +x json-tool
```

### Option 3: Add to PATH

```bash
ln -s $(pwd)/json-tool ~/.local/bin/json-tool
```

**Requirements**: Python 3.8+

---

## 📖 Usage

### `read` — Read a JSON node

```bash
json-tool read <file> <path>
```

```bash
# Read nested object field
json-tool read config.json "database.host"
# → "localhost"

# Read array element
json-tool read users.json "users[0].name"
# → "Alice"

# Read last array element
json-tool read data.json "items[-1].id"
# → 42

# Read entire array
json-tool read data.json "users"
# → [{"name": "Alice"}, {"name": "Bob"}]
```

**Alias**: `get`

---

### `edit` — Edit a JSON node

```bash
json-tool edit <file> <path> <value>
```

```bash
# Update boolean
json-tool edit config.json "debug" true

# Update number
json-tool edit config.json "database.port" 3306

# Update string
json-tool edit config.json "app.name" "\"MyApp\""

# Update with JSON object
json-tool edit config.json "database.options" '{"ssl": true, "timeout": 30}'

# Update array element
json-tool edit users.json "users[0].role" "\"admin\""
```

**Alias**: `set`

---

### `delete` — Delete a JSON node

```bash
json-tool delete <file> <path>
```

```bash
# Delete object key
json-tool delete config.json "deprecated_key"

# Delete array element
json-tool delete users.json "users[2]"
```

**Aliases**: `del`, `rm`

---

### `add` — Add/insert a JSON node

```bash
json-tool add <file> <path> <value>
```

```bash
# Add new field (auto-creates parent dicts)
json-tool add config.json "new_section.feature" "enabled"

# Append to array
json-tool add users.json "users[3]" '{"name": "Charlie"}'

# Create new file with structure
json-tool add new-config.json "app.name" "MyApp"
```

---

### `type` — Show the type of a node

```bash
json-tool type <file> <path>
```

```bash
json-tool type config.json "debug"
# → boolean

json-tool type data.json "users"
# → list
```

---

## 🔑 Path Syntax

| Path | Meaning |
|------|---------|
| `users[0].name` | First user's name field |
| `config.database.host` | Nested object traversal |
| `data.items[-1]` | Last element of items array |
| `[1]` | Root array second element |
| `[1][2]` | Root array second element's third value |

---

## 💡 Value Type Auto-Detection

| Input | Parsed As |
|-------|-----------|
| `42` | number (int) |
| `3.14` | number (float) |
| `true` / `false` | boolean |
| `null` | null |
| `"hello"` | string (with quotes) |
| `{"a": 1}` | JSON object |
| `[1, 2, 3]` | JSON array |
| `anything_else` | string literal |

---

## 🤖 For AI Agents

### Recommended Workflow

```bash
# 1. Check current value
json-tool read config.json "database.host"

# 2. Update only what you need
json-tool edit config.json "database.port" 3306

# 3. Verify the change
json-tool read config.json "database"
```

### Agent Configuration

Add this to your agent's `TOOLS.md` or system prompt:

```markdown
## json-tool
- **Path**: `~/.local/bin/json-tool` (or wherever you installed it)
- **Purpose**: Path-level JSON read/edit, preferred over text-based tools
- **When to use**: Any operation involving reading or modifying JSON files
```

---

## 🛠️ Development

This is a single-file Python script with **zero dependencies**.

```bash
# Test locally
python3 json-tool read test.json "users[0].name"
```

---

## 📄 License

MIT License — feel free to use, modify, and distribute.

---

## 🙋 Contributing

Issues and PRs welcome! This is a community tool for better AI agent workflows.

**Ideas for improvement**:
- Streaming parser for large files (>100MB)
- In-place editing without full rewrite
- JSON Patch (RFC 6902) output format
- Batch operations from stdin/JSONL

---

*Built for agents, by agents.* 🤖
