# Modularizer - Implementation Plan

## Phase 1: Foundation

### 1.1 Project Setup
- [x] Initialize Cargo project
- [ ] Set up project structure (cli, analyzer, ai, refactor, patterns modules)
- [ ] Add dependencies to Cargo.toml
- [ ] Configure CI/CD (GitHub Actions for build/test)
- [ ] Set up logging (tracing crate)

### 1.2 CLI Framework
- [ ] Implement argument parsing with clap
- [ ] Define subcommands: `analyze`, `refactor`, `check`
- [ ] Configuration file loading (modularizer.toml)
- [ ] Environment variable support for API tokens

### 1.3 Project Parsing
- [ ] Cargo.toml/workspace parsing
- [ ] Directory traversal for crates and modules
- [ ] Build Project, Crate, Module data structures
- [ ] Basic metrics collection (counts, LOC)

## Phase 2: Analysis Engine

### 2.1 sw-checklist Integration
- [ ] Execute sw-checklist as subprocess
- [ ] Parse sw-checklist output
- [ ] Map violations to internal Violation type
- [ ] Execute assist command and parse results

### 2.2 AST Analysis
- [ ] Parse Rust files with syn
- [ ] Extract function, struct, enum, trait information
- [ ] Build item inventory per module
- [ ] Detect items in mod.rs/lib.rs

### 2.3 Rule Engine
- [ ] Implement threshold-based rules
- [ ] Implement structural rules (no functions in mod.rs)
- [ ] Configurable rule parameters
- [ ] Violation reporting and formatting

## Phase 3: AI Integration

### 3.1 Provider Abstraction
- [ ] Define AiProvider trait
- [ ] Implement provider selection logic
- [ ] Add retry logic and error handling
- [ ] Implement token counting

### 3.2 Ollama Provider
- [ ] HTTP client for Ollama API
- [ ] Model listing and selection
- [ ] Streaming response support
- [ ] Health check implementation

### 3.3 Cloud Providers
- [ ] OpenAI-compatible API client
- [ ] Anthropic API client
- [ ] API key management
- [ ] Rate limiting

### 3.4 Prompt Engineering
- [ ] Design prompt templates
- [ ] Context assembly (relevant code snippets)
- [ ] Response parsing and validation
- [ ] Prompt versioning system

## Phase 4: Refactoring Engine

### 4.1 Core Engine
- [ ] RefactorPlan data structure
- [ ] FileOperation execution
- [ ] Transaction-style operations (all-or-nothing)
- [ ] Dry-run mode implementation

### 4.2 Cargo.toml Management
- [ ] Add/remove dependencies
- [ ] Add workspace members
- [ ] Create new crate Cargo.toml files
- [ ] Update internal dependencies

### 4.3 Code Generation
- [ ] Module declaration generation
- [ ] Re-export statements (pub use)
- [ ] Import fixup after moves
- [ ] Visibility adjustment

## Phase 5: Patterns

### 5.1 Split Crate Pattern
- [ ] Module dependency analysis
- [ ] AI-assisted grouping suggestions
- [ ] New crate creation
- [ ] Inter-crate dependency setup

### 5.2 Split Module Pattern
- [ ] Function grouping logic
- [ ] Submodule creation
- [ ] Re-export generation
- [ ] Import updates

### 5.3 Clean Entry Files Pattern
- [ ] Item detection in mod.rs/lib.rs
- [ ] Target file determination
- [ ] Item extraction and movement
- [ ] Export preservation

### 5.4 Distribute LOC Pattern
- [ ] Logical section detection
- [ ] File splitting strategy
- [ ] Module structure updates
- [ ] Import management

## Phase 6: User Experience

### 6.1 Output Formatting
- [ ] Colored terminal output
- [ ] Progress indicators
- [ ] Summary reports
- [ ] Diff preview for changes

### 6.2 Interactive Mode
- [ ] Confirmation prompts
- [ ] Change selection (accept/reject)
- [ ] Explanation requests
- [ ] Undo capability

### 6.3 Documentation
- [ ] CLI help text
- [ ] Pattern documentation
- [ ] Configuration reference
- [ ] Examples and tutorials

## Phase 7: Testing & Quality

### 7.1 Test Infrastructure
- [ ] Test fixture projects
- [ ] Mock AI provider
- [ ] Snapshot testing for refactorings
- [ ] Integration test framework

### 7.2 Test Coverage
- [ ] Unit tests for all modules
- [ ] Integration tests for each pattern
- [ ] End-to-end tests
- [ ] Edge case coverage

### 7.3 Quality
- [ ] Clippy compliance
- [ ] rustfmt formatting
- [ ] Documentation coverage
- [ ] Error message quality

## Phase 8: Polish & Release

### 8.1 Performance
- [ ] Parallel file processing
- [ ] Caching for repeated analysis
- [ ] Memory optimization for large projects

### 8.2 Release Preparation
- [ ] Version 0.1.0 feature freeze
- [ ] README with examples
- [ ] CHANGELOG
- [ ] crates.io publishing

---

## Milestone Summary

| Milestone | Description | Target |
|-----------|-------------|--------|
| M1 | CLI + Basic Analysis | Phase 1-2 |
| M2 | AI Integration | Phase 3 |
| M3 | Core Patterns | Phase 4-5 |
| M4 | Polish & Release | Phase 6-8 |

## Dependencies

```
clap = "4"              # CLI framework
syn = "2"               # Rust parser
quote = "1"             # Code generation
proc-macro2 = "1"       # Token handling
reqwest = "0.11"        # HTTP client
tokio = "1"             # Async runtime
serde = "1"             # Serialization
serde_json = "1"        # JSON
toml = "0.8"            # TOML config
walkdir = "2"           # Directory traversal
thiserror = "1"         # Error types
anyhow = "1"            # Error handling
tracing = "0.1"         # Logging
tracing-subscriber = "0.3"
colored = "2"           # Terminal colors
indicatif = "0.17"      # Progress bars
dialoguer = "0.11"      # Interactive prompts
similar = "2"           # Diff generation
```
