# Architecture

## Overview

tts-infra-dev is a Rust/WASM Text-to-Speech infrastructure designed with compartmentalized, testable components. The architecture prioritizes AI coding agent productivity through small, focused contexts and deterministic testing.

## Core Principle: Three Interfaces, One API

Every use case can be accomplished through three interfaces:

```
+-------------+     +-------------+     +-------------+
|    CLI      |     |  Scripting  |     |   Web UI    |
| (commands)  |     |   (tests)   |     |  (browser)  |
+------+------+     +------+------+     +------+------+
       |                   |                   |
       +-------------------+-------------------+
                           |
                           v
                  +------------------+
                  |   Core API       |
                  |   (lib crate)    |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  |   Backend Proxy  |
                  |   (TTS servers)  |
                  +------------------+
```

**Benefits**:
- **CLI**: Direct command-line access for power users and automation
- **Scripting**: Programmatic API for tests to drive correct call sequences
- **Web UI**: Visual interface for interactive use

**Testing Strategy**: Tests use the scripting interface to validate API sequences. The same sequences are then verified through CLI and Web UI.

## High-Level Architecture

```
+-------------------+     +-------------------+     +-------------------+
|   E2E Layer       |     |   UI Harness      |     |   API Layer       |
|   (Playwright)    |---->|   (ui-lab)        |---->|   (Backend)       |
+-------------------+     +-------------------+     +-------------------+
        |                         |                         |
        v                         v                         v
+-------------------+     +-------------------+     +-------------------+
| e2e/tests/        |     | ui-lab/scenarios/ |     | api-tests/        |
| s001_*.spec.ts    |     | s001_*.rs         |     | s001_*.rs         |
+-------------------+     +-------------------+     +-------------------+
```

## Three-Layer Testing Architecture

### 1. API Layer (Backend)

- Rust backend(s) with clean API contracts
- reqwest-based scenario tests
- No UI dependencies
- Base URL: `http://localhost:1100`

### 2. Scenario Harness Layer (ui-lab)

- Small Yew "micro-apps" for each scenario
- Pre-configured UI state for each test case
- Route-based scenario selection: `?scenario=S001`
- Base URL: `http://localhost:1101/lab`

### 3. E2E Automation Layer (Playwright/MCP)

- One test file per scenario
- Uses `data-testid` attributes for stable selectors
- No complex navigation required
- Direct URL access to scenario state

## Workspace Structure

```
tts-infra-dev/
|-- Cargo.toml                    # [workspace]
|
|-- core/                         # Core API library (shared by all interfaces)
|   +-- src/
|       |-- lib.rs                # Public API
|       |-- client.rs             # API client for backend calls
|       |-- types.rs              # Request/response types
|       +-- scenarios/            # Scenario implementations
|           |-- mod.rs
|           |-- s001_basic_tts.rs
|           +-- ...
|
|-- cli/                          # Command-line interface
|   +-- src/
|       |-- main.rs               # CLI entry point
|       +-- commands/             # Subcommands
|           |-- mod.rs
|           |-- synthesize.rs     # tts synthesize "Hello"
|           |-- voices.rs         # tts voices --list
|           +-- ...
|
|-- backend/                      # Backend proxy server
|   +-- src/
|       |-- main.rs
|       |-- api/                  # HTTP endpoints
|       |-- tts/                  # TTS provider integrations
|       +-- proxy/                # Multi-backend proxy
|
|-- scripting/                    # Scripting/test interface
|   +-- src/
|       |-- lib.rs                # Scripting API
|       +-- scenarios/            # Scenario test implementations
|           |-- s001_basic_tts.rs
|           +-- ...
|   +-- tests/
|       +-- integration.rs
|
|-- ui-app/                       # Production Yew application
|   +-- src/
|       |-- main.rs
|       |-- app.rs
|       |-- routes.rs
|       +-- components/
|
|-- ui-lab/                       # Scenario harness UIs
|   |-- src/
|   |   |-- main.rs               # Router + ?scenario=...
|   |   +-- scenarios/
|   |       |-- s001_basic_tts.rs
|   |       +-- ...
|   +-- static/
|       +-- index.html
|
|-- e2e/                          # Playwright test suite
|   |-- package.json
|   |-- playwright.config.ts
|   +-- tests/
|       |-- s001_basic_tts.spec.ts
|       +-- ...
|
|-- docs/
|   |-- architecture.md           # This file
|   |-- prd.md                    # Product requirements
|   |-- design.md                 # Design decisions
|   |-- plan.md                   # Implementation plan
|   |-- status.md                 # Current status
|   |-- scenarios.md              # Scenario manifest
|   |-- ports.md                  # Port assignments
|   +-- ai_playbook.md            # AI agent guidelines
|
+-- scripts/
    |-- run_backend.sh
    |-- run_ui_lab.sh
    |-- run_tests.sh
    +-- run_e2e.sh
```

## Port Assignments

| Service       | Port | Purpose                    |
|---------------|------|----------------------------|
| Backend API   | 1100 | Main TTS API               |
| UI Lab        | 1101 | Scenario harness           |
| UI App        | 1102 | Production UI (dev server) |
| Mock Backend  | 1109 | Stub service for isolation |

## Key Design Decisions

### 1. Three Interfaces, One Core API

Every use case is accessible via:
- **CLI**: `tts synthesize "Hello world" --voice=en-us`
- **Scripting**: `client.synthesize("Hello world", Some("en-us")).await?`
- **Web UI**: Form with text input and voice dropdown

All interfaces use the same `core` crate for API logic.

### 2. Scenario-First Testing

Each feature is defined as a scenario (S001, S002, etc.) with:
- Unique ID and description
- Expected API call sequence
- Dedicated test files across all layers
- Scripting tests drive the correct call order

### 3. Separate UI Lab from Production UI

The `ui-lab` provides:
- Deterministic entry points for each scenario
- Simplified DOM for AI agent interaction
- Pre-configured state via URL parameters

### 4. AI-Friendly DOM Structure

All critical controls have `data-testid` attributes:
```html
<select data-testid="voice-select">...</select>
<button data-testid="synthesize-btn">Synthesize</button>
```

### 5. Small, Focused Contexts

Each scenario involves only:
- One scripting test file (source of truth)
- One CLI command (same logic)
- One UI lab component
- One Playwright test
- Minimal documentation footprint

## Data Flow

### TTS Request Flow

```
+-------------+     +-------------+     +-------------+
|    CLI      |     |  Scripting  |     |   Web UI    |
+------+------+     +------+------+     +------+------+
       |                   |                   |
       +-------------------+-------------------+
                           |
                           v
                  +------------------+
                  |   Core API       |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  |   Backend Proxy  |
                  +--------+---------+
                           |
                  +--------+---------+
                  |        |         |
                  v        v         v
              OpenAI  ElevenLabs   Local
```

### Testing Flow

```
1. Scripting Test (scripting/)
   - Source of truth for API sequences
   - Validates correct call order
   - Fast feedback loop

2. CLI Verification (cli/)
   - Same logic as scripting
   - Command-line accessible
   - Useful for debugging

3. UI Lab Scenario (ui-lab/)
   - Component renders correctly
   - State management works
   - API calls match scripting tests

4. E2E Test (Playwright)
   - Full user journey via Web UI
   - Direct URL access
   - Stable selectors
```

## Technology Stack

| Layer           | Technology              |
|-----------------|-------------------------|
| Core API        | Rust (lib crate)        |
| CLI             | Rust, clap              |
| Backend         | Rust, Axum/Actix        |
| Scripting/Tests | Rust, reqwest           |
| UI Framework    | Yew (Rust/WASM)         |
| UI Testing      | wasm-bindgen-test       |
| E2E Testing     | Playwright via MCP      |
| State Mgmt      | Yew functional hooks    |
| Build           | cargo, wasm-pack        |
| CI/CD           | GitHub Actions          |

## Security Considerations

- API authentication via bearer tokens
- CORS configuration for UI-backend communication
- No secrets in client-side code
- Environment-based configuration

## Scalability Notes

- Backend designed for horizontal scaling
- Proxy layer enables multi-provider failover
- UI components are stateless where possible
- Database queries optimized for read-heavy workload
