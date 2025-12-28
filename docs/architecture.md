# Modularizer Architecture

## Overview

Modularizer is a Rust CLI tool that analyzes and refactors codebases to improve modularity. It integrates with `sw-checklist` for code quality analysis and uses AI (local Ollama or cloud providers) to suggest and apply intelligent refactoring patterns.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         CLI Interface                            │
│                    (clap argument parsing)                       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                       Core Orchestrator                          │
│              (coordinates analysis and refactoring)              │
└─────────────────────────────────────────────────────────────────┘
          │                     │                      │
          ▼                     ▼                      ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────┐
│   Analyzer      │  │   AI Engine     │  │   Refactoring       │
│   Module        │  │   Module        │  │   Engine            │
├─────────────────┤  ├─────────────────┤  ├─────────────────────┤
│ - sw-checklist  │  │ - Ollama client │  │ - Pattern registry  │
│ - Code metrics  │  │ - Cloud APIs    │  │ - Code transforms   │
│ - AST parsing   │  │ - Prompt mgmt   │  │ - File operations   │
└─────────────────┘  └─────────────────┘  └─────────────────────┘
          │                     │                      │
          └─────────────────────┼──────────────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                      Pattern Library                             │
│            (extensible refactoring pattern registry)             │
└─────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. CLI Interface
- Built with `clap` for argument parsing
- Accepts project repo path as primary input
- Configuration for AI provider selection (Ollama/Cloud)
- API token management
- Verbosity and dry-run options

### 2. Analyzer Module
- **sw-checklist Integration**: Runs sw-checklist and assist tools, parses results
- **Code Metrics**: Collects metrics on crates, modules, functions, LOC
- **AST Parsing**: Uses `syn` crate for Rust AST analysis
- **Violation Detection**: Identifies rule violations (e.g., functions in mod.rs/lib.rs)

### 3. AI Engine
- **Provider Abstraction**: Common interface for different AI backends
- **Ollama Support**: Local LLM inference via Ollama API
- **Cloud Provider Support**: OpenAI, Anthropic, etc. via API tokens
- **Prompt Management**: Template-based prompts for different refactoring tasks
- **Context Building**: Assembles relevant code context for AI analysis

### 4. Refactoring Engine
- **Pattern Executor**: Applies registered refactoring patterns
- **Code Generator**: Generates new files, modules, and crate structures
- **Safe Transforms**: Preview changes before applying, with rollback support
- **Cargo.toml Management**: Updates dependencies and workspace configurations

### 5. Pattern Library
- **Extensible Registry**: Patterns can be added via configuration or plugins
- **Pattern Types**:
  - Structural (split components, crates, modules)
  - Code organization (distribute LOC, extract functions)
  - Design patterns (introduce traits, builders, etc.)
  - Macro generation (derive macros, declarative macros)

## Data Flow

1. **Input**: User provides project path and configuration
2. **Analysis**: Run sw-checklist, collect metrics, parse AST
3. **Planning**: AI analyzes violations, suggests refactoring plan
4. **Preview**: Show proposed changes to user for approval
5. **Execute**: Apply refactoring patterns to codebase
6. **Verify**: Re-run sw-checklist to confirm improvements

## Directory Structure

```
modularizer/
├── src/
│   ├── main.rs              # Entry point, CLI setup
│   ├── cli/                  # CLI argument definitions
│   ├── analyzer/             # Code analysis components
│   │   ├── metrics.rs
│   │   ├── sw_checklist.rs
│   │   └── ast.rs
│   ├── ai/                   # AI integration
│   │   ├── provider.rs       # Provider trait
│   │   ├── ollama.rs
│   │   └── cloud.rs
│   ├── refactor/             # Refactoring engine
│   │   ├── engine.rs
│   │   ├── transforms.rs
│   │   └── cargo.rs
│   └── patterns/             # Pattern implementations
│       ├── registry.rs
│       ├── split_crate.rs
│       ├── split_module.rs
│       └── ...
├── patterns/                 # User-defined pattern configs (TOML/YAML)
└── prompts/                  # AI prompt templates
```

## Technology Stack

- **Language**: Rust 2021 edition
- **CLI Framework**: clap v4
- **AST Parsing**: syn, quote, proc-macro2
- **HTTP Client**: reqwest (for AI API calls)
- **Serialization**: serde, serde_json, toml
- **Async Runtime**: tokio
- **File System**: walkdir, fs_extra
- **Error Handling**: thiserror, anyhow

## Extension Points

1. **New AI Providers**: Implement the `AiProvider` trait
2. **New Patterns**: Add pattern structs implementing `RefactorPattern` trait
3. **Custom Rules**: Define rules in configuration files
4. **Prompt Templates**: Add/modify prompts in the prompts directory
