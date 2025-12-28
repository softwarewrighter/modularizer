# Modularizer - Technical Design Document

## 1. Core Abstractions

### 1.1 Project Model

```rust
/// Represents a Rust project or workspace
pub struct Project {
    pub root_path: PathBuf,
    pub workspace: Option<Workspace>,
    pub crates: Vec<Crate>,
    pub metrics: ProjectMetrics,
}

/// Workspace configuration from Cargo.toml
pub struct Workspace {
    pub members: Vec<String>,
    pub exclude: Vec<String>,
}

/// A single crate within the project
pub struct Crate {
    pub name: String,
    pub path: PathBuf,
    pub crate_type: CrateType,
    pub modules: Vec<Module>,
    pub metrics: CrateMetrics,
}

/// A module within a crate
pub struct Module {
    pub name: String,
    pub path: PathBuf,
    pub module_type: ModuleType,
    pub items: Vec<Item>,
    pub submodules: Vec<Module>,
    pub metrics: ModuleMetrics,
}

/// Items that can exist in a module
pub enum Item {
    Function(FunctionInfo),
    Struct(StructInfo),
    Enum(EnumInfo),
    Trait(TraitInfo),
    Impl(ImplInfo),
    Const(ConstInfo),
    Static(StaticInfo),
    TypeAlias(TypeAliasInfo),
    Macro(MacroInfo),
}
```

### 1.2 Metrics

```rust
pub struct ProjectMetrics {
    pub total_crates: usize,
    pub total_modules: usize,
    pub total_functions: usize,
    pub total_loc: usize,
}

pub struct CrateMetrics {
    pub module_count: usize,
    pub function_count: usize,
    pub loc: usize,
    pub public_api_count: usize,
}

pub struct ModuleMetrics {
    pub function_count: usize,
    pub struct_count: usize,
    pub loc: usize,
    pub complexity: u32,
}
```

### 1.3 Violations

```rust
pub struct Violation {
    pub rule: Rule,
    pub location: Location,
    pub severity: Severity,
    pub message: String,
    pub suggestion: Option<String>,
}

pub enum Rule {
    TooManyCrates { component: String, count: usize, max: usize },
    TooManyModules { crate_name: String, count: usize, max: usize },
    TooManyFunctions { module: String, count: usize, max: usize },
    TooManyLoc { file: PathBuf, count: usize, max: usize },
    FunctionInModRs { function: String, file: PathBuf },
    FunctionInLibRs { function: String, file: PathBuf },
    StructInModRs { struct_name: String, file: PathBuf },
    StructInLibRs { struct_name: String, file: PathBuf },
    SwChecklistViolation { code: String, message: String },
}
```

## 2. AI Provider Interface

```rust
#[async_trait]
pub trait AiProvider: Send + Sync {
    /// Get the provider name
    fn name(&self) -> &str;

    /// Generate a completion given a prompt
    async fn complete(&self, request: CompletionRequest) -> Result<CompletionResponse>;

    /// Check if the provider is available
    async fn health_check(&self) -> Result<bool>;
}

pub struct CompletionRequest {
    pub system_prompt: Option<String>,
    pub messages: Vec<Message>,
    pub max_tokens: Option<u32>,
    pub temperature: Option<f32>,
}

pub struct CompletionResponse {
    pub content: String,
    pub tokens_used: TokenUsage,
    pub model: String,
}

/// Ollama implementation
pub struct OllamaProvider {
    base_url: String,
    model: String,
    client: reqwest::Client,
}

/// OpenAI-compatible implementation (works with OpenAI, Anthropic, etc.)
pub struct CloudProvider {
    base_url: String,
    api_key: String,
    model: String,
    client: reqwest::Client,
}
```

## 3. Refactoring Pattern Interface

```rust
pub trait RefactorPattern: Send + Sync {
    /// Pattern identifier
    fn id(&self) -> &str;

    /// Human-readable description
    fn description(&self) -> &str;

    /// Check if this pattern applies to the given violation
    fn matches(&self, violation: &Violation) -> bool;

    /// Analyze and generate a refactoring plan
    fn plan(&self, project: &Project, violation: &Violation) -> Result<RefactorPlan>;

    /// Execute the refactoring
    fn execute(&self, plan: &RefactorPlan) -> Result<RefactorResult>;
}

pub struct RefactorPlan {
    pub pattern_id: String,
    pub description: String,
    pub operations: Vec<FileOperation>,
    pub cargo_changes: Vec<CargoChange>,
    pub estimated_impact: Impact,
}

pub enum FileOperation {
    CreateFile { path: PathBuf, content: String },
    DeleteFile { path: PathBuf },
    ModifyFile { path: PathBuf, changes: Vec<TextChange> },
    MoveFile { from: PathBuf, to: PathBuf },
    CreateDirectory { path: PathBuf },
}

pub struct TextChange {
    pub start_line: usize,
    pub end_line: usize,
    pub new_content: String,
}

pub enum CargoChange {
    AddDependency { crate_name: String, dep: Dependency },
    RemoveDependency { crate_name: String, dep_name: String },
    AddWorkspaceMember { member: String },
    CreateCrate { name: String, path: PathBuf },
}
```

## 4. Built-in Patterns

### 4.1 Split Crate Pattern

Triggers when a crate has too many modules. Creates new crates and redistributes modules.

```rust
pub struct SplitCratePattern {
    threshold: usize,
}

impl SplitCratePattern {
    /// Analyze module dependencies to find natural split points
    fn find_split_points(&self, crate_info: &Crate) -> Vec<ModuleGroup>;

    /// Use AI to suggest logical groupings
    async fn ai_suggest_grouping(&self, ai: &dyn AiProvider, crate_info: &Crate)
        -> Result<Vec<ModuleGroup>>;
}
```

### 4.2 Split Module Pattern

Triggers when a module has too many functions. Creates submodules and redistributes functions.

```rust
pub struct SplitModulePattern {
    threshold: usize,
}

impl SplitModulePattern {
    /// Group functions by similarity/purpose
    fn group_functions(&self, module: &Module) -> Vec<FunctionGroup>;

    /// Generate new module structure
    fn generate_submodules(&self, groups: Vec<FunctionGroup>) -> Vec<NewModule>;
}
```

### 4.3 Clean Entry Files Pattern

Moves functions and structs out of mod.rs and lib.rs files.

```rust
pub struct CleanEntryFilesPattern;

impl CleanEntryFilesPattern {
    /// Find items that should be moved
    fn find_movable_items(&self, module: &Module) -> Vec<MovableItem>;

    /// Determine target files for each item
    fn plan_moves(&self, items: Vec<MovableItem>) -> Vec<ItemMove>;
}
```

### 4.4 Distribute LOC Pattern

Splits large files into smaller, focused files.

```rust
pub struct DistributeLocPattern {
    max_loc: usize,
}

impl DistributeLocPattern {
    /// Find logical split points in a large file
    fn find_split_points(&self, file: &SourceFile) -> Vec<SplitPoint>;

    /// Generate new file structure
    fn plan_distribution(&self, file: &SourceFile, splits: Vec<SplitPoint>)
        -> Vec<NewFile>;
}
```

## 5. Prompt Templates

### 5.1 Refactoring Suggestion Prompt

```
You are analyzing a Rust codebase for modularity improvements.

## Current Structure
{project_structure}

## Violations Found
{violations}

## Metrics
{metrics}

Based on this analysis, suggest refactoring strategies to improve modularity.
For each suggestion:
1. Identify the target (crate/module/file)
2. Describe the proposed change
3. List affected files
4. Note any potential risks

Focus on:
- Logical grouping of related functionality
- Clear separation of concerns
- Maintaining public API stability
```

### 5.2 Module Grouping Prompt

```
Analyze these Rust modules and suggest logical groupings:

## Modules
{module_list}

## Dependencies Between Modules
{dependency_graph}

Group these modules into 2-4 cohesive crates. For each group:
1. Suggested crate name
2. Modules to include
3. Rationale for grouping
4. Public API boundaries
```

## 6. Error Handling

```rust
#[derive(thiserror::Error, Debug)]
pub enum ModularizerError {
    #[error("Project not found at path: {0}")]
    ProjectNotFound(PathBuf),

    #[error("Failed to parse Cargo.toml: {0}")]
    CargoParseError(#[from] toml::de::Error),

    #[error("Failed to parse Rust source: {path}")]
    ParseError { path: PathBuf, error: syn::Error },

    #[error("AI provider error: {0}")]
    AiError(String),

    #[error("Refactoring failed: {0}")]
    RefactorError(String),

    #[error("sw-checklist execution failed: {0}")]
    SwChecklistError(String),

    #[error("IO error: {0}")]
    IoError(#[from] std::io::Error),
}
```

## 7. Testing Strategy

### 7.1 Unit Tests
- Parser tests with sample Rust files
- Metrics calculation tests
- Pattern matching tests
- AI prompt generation tests

### 7.2 Integration Tests
- Full refactoring flows on test projects
- sw-checklist integration
- AI provider mocking

### 7.3 Test Fixtures
```
tests/
├── fixtures/
│   ├── simple_crate/        # Single crate, few modules
│   ├── bloated_crate/       # Crate with too many modules
│   ├── large_module/        # Module with too many functions
│   ├── dirty_mod_rs/        # mod.rs with functions
│   └── workspace/           # Multi-crate workspace
```

## 8. Security Considerations

- API tokens stored securely (environment variables, not config files)
- No code execution from AI responses
- Sandboxed file operations (only within project directory)
- Git status checks before destructive operations
