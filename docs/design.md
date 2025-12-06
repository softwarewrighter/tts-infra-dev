# Design Document

## Overview

This document captures key design decisions for tts-infra-dev, providing rationale and trade-offs for architectural choices.

## Design Principles

1. **Three Interfaces, One API**: CLI, Scripting, and Web UI share a common core
2. **Scenario Isolation**: Each test scenario is independent with its own entry point
3. **Scripting First**: Tests use scripting interface; CLI and UI follow same patterns
4. **AI Ergonomics**: Small contexts, stable selectors, explicit checklists
5. **Minimal JavaScript**: All business logic in Rust/WASM
6. **Determinism**: No flaky tests, reproducible state

## Key Design Decisions

### DD1: Three Interfaces Architecture

**Decision**: Every use case is accessible through three interfaces: CLI, Scripting, and Web UI, all sharing a common `core` crate.

**Rationale**:
- Different users prefer different interfaces (power users: CLI, testers: scripting, casual users: Web UI)
- Tests can drive the same logic programmatically via scripting
- CLI provides quick debugging and automation hooks
- Web UI can be built incrementally, with scenarios unified later

**Trade-offs**:
- (+) Maximum flexibility for users
- (+) Tests validate the exact same logic used by CLI and UI
- (+) Easy to debug via CLI when UI has issues
- (-) Three codepaths to maintain (though sharing core logic)
- (-) Slightly more initial setup

**Implementation**:
```
core/           # Shared logic: types, client, scenarios
  +-- src/lib.rs

cli/            # Uses core crate
  +-- src/main.rs (clap-based)

scripting/      # Uses core crate
  +-- src/lib.rs (test-friendly API)

ui-app/         # Uses core crate (via wasm-bindgen)
  +-- src/main.rs
```

**Example - S001 Basic TTS**:
```rust
// core/src/scenarios/s001_basic_tts.rs
pub async fn basic_tts(client: &Client, text: &str) -> Result<SynthesizeResponse> {
    client.health().await?;
    client.synthesize(text, None).await
}

// cli/src/commands/synthesize.rs
pub async fn run(args: SynthesizeArgs) -> Result<()> {
    let client = Client::new(&args.backend_url);
    let result = core::scenarios::basic_tts(&client, &args.text).await?;
    println!("{}", result.audio_url);
    Ok(())
}

// scripting/tests/s001_basic_tts.rs
#[tokio::test]
async fn test_s001_basic_tts() {
    let client = Client::new("http://localhost:1100");
    let result = core::scenarios::basic_tts(&client, "Hello").await.unwrap();
    assert!(!result.audio_url.is_empty());
}
```

### DD2: Separate UI Lab from Production UI

**Decision**: Create a dedicated `ui-lab` crate for testing scenarios, separate from the production `ui-app`.

**Rationale**:
- Production UI has complex navigation that confuses AI agents
- Test scenarios need pre-configured state
- UI Lab can have simplified DOM without affecting production aesthetics
- Enables independent evolution of testing infrastructure

**Trade-offs**:
- (+) Clean separation of concerns
- (+) AI agents work with small, focused codebase
- (-) Some code duplication between ui-lab and ui-app
- (-) Must keep shared components synchronized

**Implementation**:
```rust
// ui-lab/src/main.rs
fn main() {
    yew::Renderer::<LabRoot>::new().render();
}

// ui-lab/src/lab_root.rs
#[function_component(LabRoot)]
fn lab_root() -> Html {
    let query = get_query_string();
    let scenario_id = parse_scenario(&query);

    html! {
        <>
            <header data-testid="scenario-title">
                { format!("Scenario: {scenario_id}") }
            </header>
            { render_scenario(&scenario_id) }
        </>
    }
}
```

### DD3: URL-Based Scenario Selection

**Decision**: Use query parameters (?scenario=S001) rather than separate HTML files per scenario.

**Rationale**:
- Single WASM bundle (simpler build)
- Scenario switching via URL (simple for AI agents)
- Centralized scenario routing in one match statement
- Easy to add new scenarios

**Trade-offs**:
- (+) Single entry point for all scenarios
- (+) Simpler build configuration
- (-) All scenario code bundled together (larger WASM)
- (-) Slight routing overhead

**URL Format**:
```
http://localhost:1101/lab?scenario=S001
http://localhost:1101/lab?scenario=S001&ai_mode=1
http://localhost:1101/lab?scenario=S006&backend_down=true
```

### DD4: data-testid Convention

**Decision**: All interactive UI elements have `data-testid` attributes following a consistent naming scheme.

**Rationale**:
- Stable selectors that don't break with styling changes
- Easy for Playwright/MCP to locate elements
- Self-documenting test targets
- Works with any CSS framework

**Naming Convention**:
```
{element-type}-{purpose}
```

**Examples**:
```html
<input data-testid="text-input" />
<select data-testid="voice-select">
  <option data-testid="voice-option-en-us">English (US)</option>
</select>
<button data-testid="synthesize-btn">Synthesize</button>
<div data-testid="audio-player">...</div>
<p data-testid="error-message">...</p>
```

**Playwright Usage**:
```typescript
await page.getByTestId('text-input').fill('Hello world');
await page.getByTestId('synthesize-btn').click();
await expect(page.getByTestId('audio-player')).toBeVisible();
```

### DD5: Scenario Manifest as Single Source of Truth

**Decision**: Maintain a `docs/scenarios.md` table that defines all scenarios with their API sequences.

**Rationale**:
- One place to understand all test cases
- Prevents divergence between API tests, UI lab, and E2E tests
- Easy reference for AI agents starting a task
- Can be auto-generated from code annotations (future)

**Format**:
```markdown
| ID   | Name           | CLI Command                    | URL                  | API Sequence              |
|------|----------------|--------------------------------|----------------------|---------------------------|
| S001 | Basic TTS      | tts synthesize "Hello"         | /lab?scenario=S001   | health -> synthesize      |
| S002 | Voice Select   | tts synthesize "Hi" --voice=X  | /lab?scenario=S002   | health -> voices -> synth |
```

### DD6: Scripting-Based Testing (reqwest)

**Decision**: Use Rust's reqwest library for API tests instead of curl scripts or other tools.

**Rationale**:
- Same language as backend (consistent tooling)
- Type-safe request/response handling
- Can share types with api-model crate
- Integrated with cargo test workflow
- Easy to maintain as Rust code

**Trade-offs**:
- (+) Type safety and compile-time checks
- (+) Easy refactoring with IDE support
- (-) More verbose than curl for simple requests
- (-) Requires Rust knowledge to write tests

**Structure**:
```rust
// api-tests/src/scenarios/s001_basic_tts.rs
pub async fn run_s001_basic_tts(base_url: &str) -> anyhow::Result<()> {
    let client = reqwest::Client::new();

    // Step 1: Health check
    let resp = client
        .get(format!("{base_url}/health"))
        .send()
        .await?
        .error_for_status()?;

    // Step 2: Synthesize
    let synth_resp: SynthesizeResponse = client
        .post(format!("{base_url}/synthesize"))
        .json(&SynthesizeRequest {
            text: "Hello world".to_string()
        })
        .send()
        .await?
        .error_for_status()?
        .json()
        .await?;

    assert!(!synth_resp.audio_url.is_empty());
    Ok(())
}
```

### DD6: AI Mode Flag

**Decision**: Support an optional `?ai_mode=1` query parameter that simplifies the DOM for AI agent interaction.

**Rationale**:
- Some UI components (fancy dropdowns, modals) are hard for AI to interact with
- AI mode replaces complex components with simple HTML equivalents
- Production UI remains visually polished
- Test reliability improves significantly

**Transformations**:
| Production UI        | AI Mode Replacement     |
|----------------------|-------------------------|
| Custom dropdown      | Native `<select>`       |
| Modal dialog         | Inline form             |
| Tab panels           | Sequential sections     |
| Animated transitions | Instant state changes   |

**Implementation**:
```rust
#[function_component(VoiceSelector)]
fn voice_selector(props: &Props) -> Html {
    let ai_mode = use_context::<AiModeContext>();

    if ai_mode.enabled {
        html! {
            <select data-testid="voice-select">
                { for props.voices.iter().map(render_option) }
            </select>
        }
    } else {
        html! {
            <FancyDropdown ... />
        }
    }
}
```

### DD8: Multi-Provider Proxy Architecture

**Decision**: Backend implements a proxy layer that abstracts multiple TTS providers (OpenAI, ElevenLabs, local, etc.).

**Rationale**:
- Avoid vendor lock-in
- Enable fallback when a provider is down
- Support different providers for different use cases (quality vs. cost)
- Simplify UI - it only talks to our API

**Interface**:
```rust
trait TtsProvider {
    async fn synthesize(&self, request: SynthesizeRequest)
        -> Result<SynthesizeResponse>;
    async fn list_voices(&self) -> Result<Vec<Voice>>;
    fn provider_id(&self) -> &str;
}
```

**Configuration**:
```toml
# config.toml
[providers]
default = "openai"
fallback = ["elevenlabs", "local"]

[providers.openai]
api_key_env = "OPENAI_API_KEY"
model = "tts-1"

[providers.elevenlabs]
api_key_env = "ELEVENLABS_API_KEY"
```

### DD9: Shared Core Crate

**Decision**: Create a shared `core` crate for types, client, and scenario logic used across all interfaces.

**Rationale**:
- Single source of truth for API types AND logic
- CLI, Scripting, and UI all use the same code
- Compile-time validation of request/response shapes
- Easier refactoring when API changes
- Tests validate the exact same logic users run

**Structure**:
```rust
// core/src/lib.rs
pub mod types;
pub mod client;
pub mod scenarios;

pub use types::*;
pub use client::Client;

// core/src/types.rs
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
pub struct SynthesizeRequest {
    pub text: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub voice_id: Option<String>,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub format: Option<AudioFormat>,
}

#[derive(Debug, Serialize, Deserialize)]
pub struct SynthesizeResponse {
    pub audio_url: String,
    pub duration_ms: u64,
    pub provider: String,
}

// core/src/client.rs
pub struct Client {
    base_url: String,
    http: reqwest::Client,
}

impl Client {
    pub fn new(base_url: &str) -> Self { ... }
    pub async fn health(&self) -> Result<()> { ... }
    pub async fn synthesize(&self, text: &str, voice: Option<&str>) -> Result<SynthesizeResponse> { ... }
}
```

## Component Design

### Core Crate (Shared by All Interfaces)

```
core/
|-- src/
    |-- lib.rs            # Public API exports
    |-- types.rs          # Request/response types
    |-- client.rs         # API client (reqwest-based)
    |-- error.rs          # Error types
    +-- scenarios/
        |-- mod.rs
        |-- s001_basic_tts.rs
        |-- s002_voice_selection.rs
        +-- ...
```

### CLI Components

```
cli/
|-- src/
    |-- main.rs           # Entry point (clap)
    +-- commands/
        |-- mod.rs
        |-- health.rs     # tts health
        |-- synthesize.rs # tts synthesize "text"
        |-- voices.rs     # tts voices
        +-- ...
```

### Backend Components

```
backend/
|-- src/
    |-- main.rs           # Server setup, routing
    |-- api/
    |   |-- mod.rs
    |   |-- health.rs     # GET /health
    |   |-- synthesize.rs # POST /synthesize
    |   +-- voices.rs     # GET /voices
    |-- tts/
    |   |-- mod.rs
    |   |-- provider.rs   # TtsProvider trait
    |   |-- openai.rs     # OpenAI implementation
    |   |-- elevenlabs.rs # ElevenLabs implementation
    |   +-- local.rs      # Local TTS implementation
    +-- proxy/
        |-- mod.rs
        +-- selector.rs   # Provider selection logic
```

### Scripting Components

```
scripting/
|-- src/
    |-- lib.rs            # Re-exports core for easy test use
    +-- scenarios/        # Optional scenario helpers
|-- tests/
    |-- s001_basic_tts.rs
    |-- s002_voice_selection.rs
    +-- ...
```

### UI Lab Components

```
ui-lab/
|-- src/
    |-- main.rs
    |-- lab_root.rs       # Scenario router
    |-- context.rs        # AiModeContext, etc.
    +-- scenarios/
        |-- mod.rs
        |-- s001_basic_tts.rs
        |-- s002_voice_selection.rs
        +-- ...
```

## Error Handling Strategy

### API Errors

```rust
#[derive(Debug, Serialize)]
pub struct ApiError {
    pub code: String,
    pub message: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub details: Option<serde_json::Value>,
}

// HTTP status codes:
// 400 - Invalid input
// 401 - Unauthorized
// 429 - Rate limited
// 500 - Provider error
// 503 - All providers down
```

### UI Error Display

```rust
#[function_component(ErrorMessage)]
fn error_message(props: &ErrorProps) -> Html {
    html! {
        <div data-testid="error-message" class="error">
            <span data-testid="error-code">{ &props.code }</span>
            <span data-testid="error-text">{ &props.message }</span>
        </div>
    }
}
```

## State Management

### UI Lab State

Each scenario manages its own state using Yew hooks:
```rust
#[function_component(S001BasicTts)]
fn s001_basic_tts() -> Html {
    let text = use_state(|| "Hello world".to_string());
    let result = use_state(|| None::<SynthesizeResponse>);
    let error = use_state(|| None::<ApiError>);
    let loading = use_state(|| false);

    // ... component logic
}
```

### No Global State in UI Lab

- Each scenario is self-contained
- No shared state between scenarios
- Simplifies testing and debugging

## Future Considerations

### Potential Enhancements

1. **Code Generation**: Generate scenario boilerplate from manifest
2. **Visual Regression**: Screenshot comparison for UI components
3. **Performance Benchmarks**: Automated timing tests
4. **Coverage Reports**: Track test coverage per scenario

### Migration Path

When production UI patterns stabilize:
1. Extract shared components to a `ui-shared` crate
2. Both ui-app and ui-lab depend on ui-shared
3. Reduce code duplication while maintaining separation
