---
name: ast-index
description: Rebuild the L1-L4 tree-sitter AST index for fast semantic search. L1=files, L2=symbols, L3=signatures, L4=dependencies. Respects .gitignore.
argument-hint: "[--level L1|L2|L3|L4] [--language typescript|python|rust|bash]"
---

# /ast-index - Rebuild AST Index

## Usage

```
/ast-index                    # rebuild all 4 layers
/ast-index --level L2         # rebuild only symbol index
/ast-index --language python  # rebuild only Python files
```

## Layers

| Layer | File | Contents |
|-------|------|----------|
| **L1** | `ast-index/L1-files.json` | File manifest: path, language, size, lines |
| **L2** | `ast-index/L2-symbols.json` | Symbols: functions, classes, types, exports |
| **L3** | `ast-index/L3-signatures.json` | Full function signatures with params/returns |
| **L4** | `ast-index/L4-dependencies.json` | Import/dependency graph per file |

## Execute

```bash
source .venv/bin/activate && python scripts/build-ast-index.py
```

If `.venv` doesn't exist:
```bash
uv venv .venv && source .venv/bin/activate
uv pip install tree-sitter tree-sitter-typescript tree-sitter-python tree-sitter-bash tree-sitter-json tree-sitter-rust tree-sitter-markdown
python scripts/build-ast-index.py
```

## Query Examples

```bash
# Find all classes
jq '.[] | select(.symbols[] | .kind == "class")' ast-index/L2-symbols.json

# Functions accepting Request
jq '.[] | .signatures[] | select(.signature | contains("Request"))' ast-index/L3-signatures.json

# What imports jade-enterprise-sdk
jq '.[] | select(.dependencies[] | .statement | contains("jade-enterprise-sdk"))' ast-index/L4-dependencies.json

# Python files over 100 lines
jq '.[] | select(.language == "python" and .lines > 100)' ast-index/L1-files.json
```

## How It Works

1. `git ls-files` collects tracked files (respects `.gitignore`)
2. Tree-sitter parses TypeScript, Python, Rust, Bash
3. AST walk (depth 3) extracts symbols, signatures, imports
4. JSON output to `ast-index/` (gitignored)
