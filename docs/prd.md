# Modularizer - Product Requirements Document

## Problem Statement

Large Rust codebases often suffer from poor modularity:
- Components accumulate too many crates
- Crates grow to contain too many modules
- Modules become bloated with too many functions
- Single files exceed reasonable LOC limits
- Utility files like `mod.rs` and `lib.rs` inappropriately contain business logic

Manual refactoring is time-consuming, error-prone, and requires deep understanding of Rust's module system. Developers need automated tooling that can analyze codebases, identify modularity issues, and apply intelligent refactoring.

## Solution

Modularizer is a CLI tool that:
1. Analyzes Rust projects using sw-checklist and custom metrics
2. Leverages AI to understand code context and suggest refactoring strategies
3. Applies refactoring patterns to improve modularity
4. Verifies improvements through re-analysis

## Target Users

- **Rust developers** maintaining large codebases
- **Tech leads** enforcing code quality standards
- **Teams** adopting sw-checklist compliance
- **Open source maintainers** improving project structure

## Features

### Core Features (MVP)

#### F1: Project Analysis
- Accept project path as CLI argument
- Run sw-checklist and parse results
- Collect metrics: crate count, module count, function count, LOC per file
- Identify rule violations

#### F2: AI Integration
- Support local Ollama server for offline/private use
- Support cloud AI providers (OpenAI, Anthropic) via API tokens
- Generate refactoring suggestions based on analysis
- Provide explanations for suggested changes

#### F3: Refactoring Patterns
- **Split Component**: Divide component when crate count exceeds threshold
- **Split Crate**: Divide crate when module count exceeds threshold
- **Split Module**: Divide module when function count exceeds threshold
- **Distribute LOC**: Split large files across multiple .rs files
- **Clean Entry Files**: Move functions/structs out of mod.rs and lib.rs

#### F4: Safe Refactoring
- Preview mode showing proposed changes
- Dry-run option for validation
- Git-aware operations (warn on uncommitted changes)
- Rollback support via git integration

### Future Features

#### F5: Design Pattern Introduction
- Suggest and implement common Rust patterns
- Builder pattern for complex structs
- Type state pattern for state machines
- Strategy pattern via traits

#### F6: Macro Generation
- Generate derive macros for common traits
- Create declarative macros for repetitive code
- Suggest procedural macros where appropriate

#### F7: Interactive Mode
- Step-by-step refactoring guidance
- Accept/reject individual changes
- AI-assisted code review

#### F8: CI/CD Integration
- GitHub Actions workflow generation
- Pre-commit hook support
- Incremental analysis for PRs

## Configuration

### CLI Arguments
```
modularizer <PROJECT_PATH> [OPTIONS]

Options:
  --ai-provider <PROVIDER>    AI provider: ollama, openai, anthropic
  --api-token <TOKEN>         API token for cloud providers
  --ollama-url <URL>          Ollama server URL (default: http://localhost:11434)
  --dry-run                   Preview changes without applying
  --verbose                   Verbose output
  --config <PATH>             Path to config file
```

### Configuration File (modularizer.toml)
```toml
[thresholds]
max_crates_per_component = 10
max_modules_per_crate = 15
max_functions_per_module = 20
max_loc_per_file = 500

[rules]
no_functions_in_mod_rs = true
no_functions_in_lib_rs = true
no_structs_in_mod_rs = true
no_structs_in_lib_rs = true

[ai]
default_provider = "ollama"
ollama_model = "codellama"
```

## Success Metrics

1. **Adoption**: Number of projects using modularizer
2. **Compliance**: Improvement in sw-checklist scores after running
3. **Efficiency**: Time saved vs manual refactoring
4. **Accuracy**: Percentage of refactorings requiring no manual fixes

## Constraints

- Must work with Rust 2018 and 2021 editions
- Must preserve code semantics (no behavioral changes)
- Must handle workspace and single-crate projects
- AI calls must be optional (offline mode available)

## Out of Scope (v1.0)

- Non-Rust language support
- IDE plugins (VS Code, IntelliJ)
- Automatic test generation
- Performance optimization refactoring
