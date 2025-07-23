# MCP Rust Template 🦀

[![Build Status](https://github.com/belyak/mcp-claude-stdio-template/actions/workflows/ci.yml/badge.svg)](https://github.com/belyak/mcp-claude-stdio-template/actions)
[![License](https://img.shields.io/github/license/belyak/mcp-claude-stdio-template)](LICENSE)
[![Maintenance](https://img.shields.io/badge/maintenance-very%20early%20beta-yellow)](VERSION)
[![GitHub stars](https://img.shields.io/github/stars/belyak/mcp-claude-stdio-template?style=social)](https://github.com/belyak/mcp-claude-stdio-template)

A comprehensive **Rust template** for building [Model Context Protocol (MCP)](https://modelcontextprotocol.io/) stdio servers, designed for seamless integration with **Claude Desktop** and other MCP clients. This template provides a complete foundation with pre-built tools, prompts, resources, and automated deployment scripts.

---

## 🚀 Features

### Core Capabilities
- **Full MCP Protocol Implementation** - Complete JSON-RPC based MCP server
- **Template System** - Uses `cargo-generate` for easy project scaffolding
- **Claude Desktop Integration** - Ready-to-use configuration and installation scripts
- **Modular Architecture** - Organized codebase with separate modules for tools, prompts, and resources
- **Signal Handling** - Graceful shutdown support with SIGINT/SIGTERM handling
- **Comprehensive Logging** - Built-in logging to `/tmp/mcp.jsonl` for debugging

### Pre-built Components
- **Tools** - Example time tool with extensible framework
- **Prompts** - Template system for dynamic prompt generation
- **Resources** - Resource management with subscription support
- **Templates** - JSON configuration files for easy customization

### Development & Deployment
- **CLI Interface** - Multiple command-line options for testing and inspection
- **Automated Testing** - Template validation and build verification
- **Installation Script** - One-command deployment to system and Claude Desktop
- **Development Tools** - Justfile with common MCP operations
- **CI/CD Ready** - GitHub Actions workflow included

---

## 📋 Requirements

- **Rust** 1.70+ with Cargo
- **macOS** (for Claude Desktop integration)
- **cargo-generate** (for template usage)

---

## 🛠️ Quick Start

### 1. Generate New Project

```bash
# Install cargo-generate if needed
cargo install cargo-generate

# Generate new MCP server from template
cargo generate --git https://github.com/belyak/mcp-gen-rs-tpl-tool-site
```

You'll be prompted for:
- **Project name** - Your MCP server's name
- **Description** - Brief description of your server
- **Keywords** - Comma-separated keywords for Cargo

### 2. Build & Test

```bash
cd your-project-name
cargo build
cargo test
```

### 3. Test MCP Functionality

```bash
# List available tools
cargo run -- --tools

# List prompts and resources
cargo run -- --prompts --resources

# Test JSON-RPC output
cargo run -- --tools --json

# Start MCP server (for integration testing)
cargo run -- --mcp
```

---

## 🔧 Architecture

### Project Structure

```
src/
├── main.rs              # Application entry point & JSON-RPC handling
├── lib.rs               # Library interface
└── mcp/
    ├── mod.rs           # MCP module constants
    ├── types.rs         # MCP protocol type definitions
    ├── utilities.rs     # Core MCP utilities & handlers
    ├── tools.rs         # Tools implementation
    ├── prompts.rs       # Prompts implementation
    ├── resources.rs     # Resources implementation
    └── templates/       # JSON configuration templates
        ├── tools.json
        ├── prompts.json
        ├── resources.json
        └── README.md
```

### Key Components

#### 1. **MCP Protocol Handler** (`main.rs`)
- JSON-RPC request/response processing
- Signal handling for graceful shutdown
- Logging and error handling
- CLI argument parsing

#### 2. **Type System** (`types.rs`)
- Complete MCP protocol type definitions
- Serialization/deserialization support
- Request/response structures

#### 3. **Tools Framework** (`tools.rs`)
- Dynamic tool registration system
- Example time tool implementation
- Extensible tool interface

#### 4. **Template System** (`templates/`)
- JSON-based configuration
- Easy customization without code changes
- Hot-reloadable definitions

---

## 🎯 Usage

### Command Line Interface

```bash
# Display available options
your-mcp-server --help

# List components
your-mcp-server --tools           # Show available tools
your-mcp-server --prompts         # Show available prompts  
your-mcp-server --resources       # Show available resources

# JSON output (for scripting)
your-mcp-server --tools --json

# Start MCP server
your-mcp-server --mcp
your-mcp-server --stdio           # Alias for --mcp
```

### Integration with Claude Desktop

#### Automatic Installation
```bash
# Build, install, and configure everything
./install_mcp.sh
```

#### Manual Configuration
Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "your-server-name": {
      "command": "/path/to/your-binary",
      "args": ["--mcp"],
      "env": {
        "API_KEY": "your-api-key-if-needed"
      }
    }
  }
}
```

### Testing MCP Operations

Using the included **Justfile**:

```bash
just ping              # Test server connectivity
just tools-list        # List available tools
just prompts-list      # List available prompts
just resources-list    # List available resources
just current-time      # Test time tool
```

---

## 🧪 Development

### Template Validation

```bash
# Validate template generates correctly
./test_template.sh
```

This script:
- ✅ Generates a new project from template
- ✅ Builds the generated project
- ✅ Runs all tests
- ✅ Provides colored progress report
- ✅ Cleans up test artifacts

### Customizing Components

#### Adding New Tools

1. **Update templates/tools.json**:
```json
{
  "name": "your_tool",
  "description": "Your tool description",
  "inputSchema": {
    "type": "object",
    "properties": {
      "param": {"type": "string", "description": "Parameter description"}
    },
    "required": ["param"]
  }
}
```

2. **Implement in tools.rs**:
```rust
pub async fn your_tool(request: YourToolRequest) -> HandlerResult<CallToolResult> {
    // Implementation
}
```

3. **Register in router**:
```rust
pub fn register_tools(router_builder: RouterBuilder) -> RouterBuilder {
    router_builder
        .append_dyn("your_tool", your_tool.into_dyn())
}
```

#### Adding Prompts & Resources

Follow similar patterns in `prompts.rs` and `resources.rs` with corresponding JSON template updates.

### Debugging

- **MCP Logs**: `tail -f ~/Library/Logs/Claude/mcp*.log`
- **Server Logs**: `/tmp/mcp.jsonl`
- **Verbose Testing**: `RUST_LOG=debug cargo test`

---

## 📦 Dependencies

### Core Dependencies
- **tokio** - Async runtime
- **serde** + **serde_json** - Serialization
- **rpc-router** - JSON-RPC routing
- **clap** - CLI argument parsing
- **chrono** - Date/time handling
- **signal-hook** - Signal handling

### Development Dependencies
- **cargo-generate** - Template generation
- **just** - Command runner

---

## 🚀 Deployment

### System Installation

```bash
# Build optimized release
cargo build --release

# Install system-wide
sudo cp target/release/your-binary /usr/local/bin/

# Or use the automated script
./install_mcp.sh
```

### Distribution

The template supports creating distributable MCP servers:

1. **Build for release**: Optimized binary with LTO and size optimization
2. **Package**: Include binary + configuration examples
3. **Document**: Usage instructions for end users

---

## 🔍 Template Variables

When generating from template:

| Variable | Description | Default |
|----------|-------------|---------|
| `project-name` | Your project name | *Required* |
| `description` | Project description | "Model Context Protocol (MCP) Rust CLI server template" |
| `keywords` | Cargo keywords | "rust,ai,mcp,cli" |

---

## 🧪 Testing & Quality Assurance

### Automated Testing
```bash
cargo test              # Unit tests
./test_template.sh      # Template validation
```

### Manual Testing
```bash
# Test CLI functionality
cargo run -- --tools --prompts --resources

# Test MCP protocol
echo '{"jsonrpc":"2.0","id":1,"method":"ping"}' | cargo run -- --mcp
```

### Performance Optimization

The template includes optimized build profiles:
- **Development**: Fast compilation with basic optimization
- **Release**: Full optimization with LTO and size reduction

---

## 📚 References & Resources

- **[MCP Specification](https://spec.modelcontextprotocol.io/)** - Official protocol documentation
- **[Model Context Protocol](https://modelcontextprotocol.io/introduction)** - Introduction and guides
- **[rpc-router](https://github.com/jeremychone/rust-rpc-router/)** - JSON-RPC routing library
- **[Zed Context Server](https://github.com/zed-industries/zed/tree/main/crates/context_server)** - Reference implementation
- **[cargo-generate](https://github.com/cargo-generate/cargo-generate)** - Template generation tool

---

## 📄 License

This project is licensed under the **Apache License 2.0**. See the [LICENSE](LICENSE) file for details.

---

## 🤝 Contributing

Contributions are welcome! This is a very early beta project, so there's plenty of room for improvement:

- 🐛 **Bug Reports** - Found an issue? Please report it
- 💡 **Feature Requests** - Have ideas for improvements?
- 📖 **Documentation** - Help improve the docs
- 🧪 **Testing** - Add more comprehensive tests
- 🔧 **Tools** - Contribute new example tools

---

## 📞 Support

- **Issues**: [GitHub Issues](https://github.com/belyak/mcp-claude-stdio-template/issues)
- **Discussions**: [GitHub Discussions](https://github.com/belyak/mcp-claude-stdio-template/discussions)
- **MCP Community**: [Official MCP Resources](https://modelcontextprotocol.io/)

---

*Built with ❤️ and 🦀 for the MCP ecosystem*
