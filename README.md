# Modularizer

A Rust CLI tool that analyzes and refactors codebases to improve modularity, using AI to intelligently apply refactoring patterns.

## Overview

Modularizer integrates with `sw-checklist` to analyze Rust projects and automatically refactor code to:
- Split components with too many crates
- Split crates with too many modules
- Split modules with too many functions
- Distribute lines of code across multiple files
- Enforce structural rules (no functions/structs in mod.rs or lib.rs)

## Features

- **Automated Analysis**: Runs sw-checklist and collects code metrics
- **AI-Powered Refactoring**: Uses local Ollama or cloud AI providers for intelligent suggestions
- **Extensible Patterns**: Add custom refactoring patterns over time
- **Safe Operations**: Preview changes before applying, with dry-run support
- **Design Patterns**: Introduce Rust idioms, macros, and design patterns

## Installation

```bash
cargo install modularizer
```

Or build from source:

```bash
git clone https://github.com/softwarewrighter/modularizer
cd modularizer
cargo build --release
```

## Usage

```bash
# Analyze a project
modularizer analyze /path/to/project

# Refactor with AI assistance (using Ollama)
modularizer refactor /path/to/project --ai-provider ollama

# Refactor using cloud AI
modularizer refactor /path/to/project --ai-provider openai --api-token $OPENAI_API_KEY

# Dry run (preview changes)
modularizer refactor /path/to/project --dry-run
```

## Configuration

Create a `modularizer.toml` in your project root:

```toml
[thresholds]
max_crates_per_component = 10
max_modules_per_crate = 15
max_functions_per_module = 20
max_loc_per_file = 500

[rules]
no_functions_in_mod_rs = true
no_functions_in_lib_rs = true

[ai]
default_provider = "ollama"
ollama_model = "codellama"
```

## Documentation

- [Architecture](docs/architecture.md) - System design and components
- [Product Requirements](docs/prd.md) - Features and specifications
- [Technical Design](docs/design.md) - Implementation details
- [Implementation Plan](docs/plan.md) - Development roadmap
- [Project Status](docs/status.md) - Current progress

## Refactoring Patterns

| Pattern | Trigger | Action |
|---------|---------|--------|
| Split Component | Too many crates | Divide into multiple components |
| Split Crate | Too many modules | Create new crates |
| Split Module | Too many functions | Create submodules |
| Distribute LOC | File too large | Split into multiple files |
| Clean Entry Files | Code in mod.rs/lib.rs | Move to dedicated modules |

## Requirements

- Rust 1.70+
- `sw-checklist` (for analysis integration)
- Ollama (optional, for local AI)
- API token (optional, for cloud AI providers)

## Contributing

Contributions welcome! Please read the documentation and open issues for discussion before submitting PRs.

## License

MIT OR Apache-2.0
