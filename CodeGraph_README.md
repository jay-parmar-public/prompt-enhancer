# CodeGraph

**A local code knowledge graph that indexes your codebase and serves it to AI agents via MCP.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

## What is CodeGraph?

CodeGraph parses your source code, builds a knowledge graph of symbols and their relationships, and exposes it to AI coding agents through the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/). When your agent needs to understand how functions connect, trace impact across files, or search for symbols — CodeGraph answers in milliseconds.

## Why CodeGraph?

- **Zero network activity** — everything runs locally, your code never leaves your machine
- **Sub-50ms queries** — SQLite + FTS5 full-text search with graph traversal
- **Auto-sync** — file watcher detects changes and re-indexes incrementally
- **Single binary** — no runtime dependencies, no Docker, no separate processes
- **10 MCP tools** — search, context building, impact analysis, callers/callees, and more

## Installation

### Option 1: Download Pre-built Binary

Download the latest release for your platform from [GitHub Releases](https://github.com/jay-parmar/codegraph/releases).

**Linux / macOS:**

```bash
chmod +x codegraph-<target>
sudo mv codegraph-<target> /usr/local/bin/codegraph
```

**Windows:**

Move `codegraph-<target>.exe` to a directory on your `PATH`.

### Option 2: Install via Cargo

Once published to crates.io:

```bash
cargo install codegraph
```

### Option 3: Build from Source

```bash
git clone https://github.com/jay-parmar/codegraph.git
cd codegraph
cargo build --release
```

The binary will be at `target/release/codegraph` (or `codegraph.exe` on Windows).

## Quick Start

```bash
# 1. Initialize CodeGraph in your project
cd /path/to/your/project
codegraph init

# 2. Start the MCP server
codegraph serve
```

Then [configure your AI agent](#mcp-client-setup) to connect to CodeGraph.

## MCP Client Setup

After installing, configure your AI agent to use CodeGraph as an MCP server.

### VS Code

Copy to `.vscode/mcp.json` in your project:

```json
{
  "servers": {
    "codegraph": {
      "type": "stdio",
      "command": "codegraph",
      "args": ["serve"]
    }
  }
}
```

### Cursor

Copy to `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "codegraph": {
      "command": "codegraph",
      "args": ["serve"]
    }
  }
}
```

### Claude Desktop

Merge into your Claude Desktop config file:

- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **Linux:** `~/.config/Claude/claude_desktop_config.json`

```json
{
  "mcpServers": {
    "codegraph": {
      "command": "codegraph",
      "args": ["serve"]
    }
  }
}
```

> **Note:** If `codegraph` is not on your `PATH`, replace `"codegraph"` with the full path to the binary (e.g., `"/usr/local/bin/codegraph"` or `"C:\\Users\\you\\.cargo\\bin\\codegraph.exe"`).

See [`configs/`](configs/) for full configuration examples with path customization comments.

## CLI Reference

### Global Options

| Option | Description |
|--------|-------------|
| `--log-level <LEVEL>` | Set log level: `trace`, `debug`, `info`, `warn`, `error` (default: `info`) |
| `--config <PATH>` | Path to `codegraph.toml` configuration file |
| `-h, --help` | Print help |
| `-V, --version` | Print version |

### Commands

#### `codegraph init`

Initialize CodeGraph for the current workspace. Creates the `.codegraph/` directory and database.

```bash
codegraph init
```

#### `codegraph index`

Rebuild the full index from scratch. Re-parses all source files and rebuilds the knowledge graph.

```bash
codegraph index
```

#### `codegraph search <QUERY>`

Search symbols by name using full-text search.

```bash
# Search for a function
codegraph search "handleRequest"

# Filter by symbol kind
codegraph search "User" --kind class

# Filter by language with a result limit
codegraph search "parse" --language typescript --limit 10
```

| Option | Description |
|--------|-------------|
| `<QUERY>` | Symbol name or search query (required) |
| `--kind <KIND>` | Filter by symbol kind (function, class, method, etc.) |
| `--language <LANGUAGE>` | Filter by programming language |
| `--limit <LIMIT>` | Maximum number of results (default: 20) |

#### `codegraph serve`

Start the MCP server on stdio transport. This is the command your AI agent calls.

```bash
codegraph serve
```

#### `codegraph status`

Display index status and statistics.

```bash
codegraph status
```

#### `codegraph impact <SYMBOL>`

Run impact analysis for a symbol. Traces callers and callees to show the full impact radius.

```bash
# Analyze impact in both directions
codegraph impact "src/server/tools.rs::handle_search"

# Upstream callers only, limited depth
codegraph impact "src/main.rs::main" --direction up --depth 3
```

| Option | Description |
|--------|-------------|
| `<SYMBOL>` | Symbol FQN to analyze (required) |
| `--direction <DIR>` | Direction: `up`, `down`, or `both` (default: `both`) |
| `--depth <N>` | Maximum traversal depth (default: 5) |

## MCP Tools

CodeGraph exposes 10 tools through the MCP protocol. All tools return structured JSON with metadata including index version and staleness information.

| Tool | Description |
|------|-------------|
| `codegraph_get_index_status` | Check indexing status and statistics |
| `codegraph_search_symbols` | Full-text symbol search with filters |
| `codegraph_resolve_symbol` | Resolve a symbol by fully qualified name |
| `codegraph_get_callers` | Find all callers of a symbol |
| `codegraph_get_callees` | Find all callees of a symbol |
| `codegraph_get_impact_analysis` | Trace the full impact radius of a symbol |
| `codegraph_get_symbol_context` | Smart context builder — returns definition, callers, callees, related types |
| `codegraph_get_file_symbols` | List all symbols in a file |
| `codegraph_entry_points` | Find entry points reachable from a symbol |
| `codegraph_get_dependencies` | Get dependencies for a symbol |

## Configuration

CodeGraph works out of the box with zero configuration. To customize behavior, create a `codegraph.toml` file in your project root.

### Options

| Section | Key | Type | Default | Description |
|---------|-----|------|---------|-------------|
| *(root)* | `log_level` | string | `"info"` | Logging level: `trace`, `debug`, `info`, `warn`, `error` |
| `[languages]` | `enabled` | string[] | `[]` | Languages to index. Empty list = auto-detect from file extensions |
| `[watcher]` | `debounce_ms` | integer | `300` | Debounce interval in milliseconds for file change events |
| `[watcher]` | `ignore_patterns` | string[] | `[]` | Additional ignore patterns (glob syntax, augments `.gitignore`) |
| `[query]` | `max_depth` | integer | `20` | Maximum traversal depth for recursive queries |
| `[query]` | `max_results` | integer | `500` | Maximum number of results per query |
| `[query]` | `context_max_size` | integer | `30720` | Maximum context response size in bytes (30 KB) |

### Example

```toml
log_level = "debug"

[languages]
enabled = ["typescript", "csharp"]

[watcher]
debounce_ms = 500
ignore_patterns = ["**/generated/**", "**/vendor/**"]

[query]
max_depth = 10
max_results = 100
context_max_size = 61440
```

## Supported Languages

| Language | Extensions | Symbol Kinds |
|----------|------------|-------------|
| TypeScript | `.ts`, `.tsx` | functions, methods, classes, interfaces, type aliases, enums, variables, constants, modules |
| C# | `.cs` | methods, classes, interfaces, enums, structs, properties, fields, namespaces |

## Architecture Overview

CodeGraph follows a four-stage pipeline:

1. **Parse** — [Tree-sitter](https://tree-sitter.github.io/) grammars extract symbols and relationships from source files
2. **Store** — SQLite database with FTS5 full-text search indexes all symbols, edges, and metadata
3. **Query** — Graph traversal engine answers callers/callees, impact analysis, and context-building queries using recursive CTEs
4. **Serve** — MCP server exposes the knowledge graph to AI agents over stdio transport

The file watcher monitors your workspace for changes and triggers incremental re-indexing automatically.

## Contributing

### Build from Source

```bash
git clone https://github.com/jay-parmar/codegraph.git
cd codegraph
cargo build
```

Requires **Rust 2024 edition** (Rust 1.85.0+).

### Run Tests

```bash
cargo test
```

### Lint and Format

```bash
cargo clippy --all-targets -- -D warnings
cargo fmt --check
```

All pull requests must pass clippy with `-D warnings` and `cargo fmt --check`.

## License

[MIT](https://opensource.org/licenses/MIT)
