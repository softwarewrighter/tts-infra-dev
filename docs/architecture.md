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

## Physical Layout

The project uses a component-based structure with strict limits enforced by `sw-checklist`:

### sw-checklist Constraints

| Metric              | Warn  | Fail  |
|---------------------|-------|-------|
| Lines per function  | >25   | >50   |
| Functions per module| >4    | >7    |
| Modules per crate   | >4    | >7    |

### Code Organization Rules

1. **No functions in lib.rs or mod.rs** - these files contain only re-exports
2. **Each .rs file is focused** - one responsibility per file
3. **Rust Edition 2024** - all crate Cargo.toml files use `edition = "2024"`

### Directory Structure

```
tts-infra-dev/
|
|-- components/
|   |
|   |-- backend/                          # TTS proxy server component
|   |   |-- Cargo.toml                    # Workspace manifest
|   |   +-- crates/
|   |       |-- backend-api/              # HTTP endpoint handlers
|   |       |   |-- Cargo.toml            # edition = "2024"
|   |       |   +-- src/
|   |       |       |-- lib.rs            # Re-exports only
|   |       |       |-- health/
|   |       |       |   |-- mod.rs        # Re-exports only
|   |       |       |   +-- handler.rs    # GET /health implementation
|   |       |       |-- synthesize/
|   |       |       |   |-- mod.rs
|   |       |       |   |-- handler.rs    # POST /synthesize
|   |       |       |   +-- validation.rs # Input validation
|   |       |       +-- voices/
|   |       |           |-- mod.rs
|   |       |           +-- handler.rs    # GET /voices
|   |       |
|   |       |-- backend-proxy/            # Provider proxy logic
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- lib.rs
|   |       |       |-- selector/
|   |       |       |   |-- mod.rs
|   |       |       |   +-- strategy.rs   # Provider selection
|   |       |       +-- fallback/
|   |       |           |-- mod.rs
|   |       |           +-- chain.rs      # Fallback chain
|   |       |
|   |       +-- backend-tts/              # TTS provider implementations
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               |-- provider/
|   |               |   |-- mod.rs
|   |               |   +-- trait_def.rs  # TtsProvider trait
|   |               |-- openai/
|   |               |   |-- mod.rs
|   |               |   +-- client.rs     # OpenAI TTS
|   |               +-- elevenlabs/
|   |                   |-- mod.rs
|   |                   +-- client.rs     # ElevenLabs TTS
|   |
|   |-- cli/                              # Command-line interface component
|   |   |-- Cargo.toml                    # Workspace manifest
|   |   +-- crates/
|   |       |-- cli-app/                  # CLI binary
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- main.rs           # Entry point (minimal)
|   |       |       +-- lib.rs            # Re-exports
|   |       |
|   |       +-- cli-commands/             # Command implementations
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               |-- health/
|   |               |   |-- mod.rs
|   |               |   +-- run.rs        # tts health
|   |               |-- synthesize/
|   |               |   |-- mod.rs
|   |               |   |-- run.rs        # tts synthesize
|   |               |   +-- args.rs       # Argument parsing
|   |               +-- voices/
|   |                   |-- mod.rs
|   |                   +-- run.rs        # tts voices
|   |
|   |-- core/                             # Shared types and client
|   |   |-- Cargo.toml                    # Workspace manifest
|   |   +-- crates/
|   |       |-- core-types/               # Request/response types
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- lib.rs
|   |       |       |-- request/
|   |       |       |   |-- mod.rs
|   |       |       |   +-- synthesize.rs
|   |       |       +-- response/
|   |       |           |-- mod.rs
|   |       |           |-- synthesize.rs
|   |       |           +-- voice.rs
|   |       |
|   |       |-- core-client/              # API client
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- lib.rs
|   |       |       |-- client/
|   |       |       |   |-- mod.rs
|   |       |       |   |-- builder.rs    # Client builder
|   |       |       |   +-- methods.rs    # API methods
|   |       |       +-- config/
|   |       |           |-- mod.rs
|   |       |           +-- settings.rs   # Client config
|   |       |
|   |       |-- core-error/               # Error types
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- lib.rs
|   |       |       +-- error/
|   |       |           |-- mod.rs
|   |       |           |-- api.rs        # API errors
|   |       |           +-- client.rs     # Client errors
|   |       |
|   |       +-- core-scenarios/           # Scenario implementations
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               |-- s001/
|   |               |   |-- mod.rs
|   |               |   +-- basic_tts.rs
|   |               +-- s002/
|   |                   |-- mod.rs
|   |                   +-- voice_select.rs
|   |
|   |-- mocks/                            # Mock implementations
|   |   |-- Cargo.toml
|   |   +-- crates/
|   |       |-- mock-backend/             # Mock backend server
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- lib.rs
|   |       |       +-- server/
|   |       |           |-- mod.rs
|   |       |           +-- handlers.rs
|   |       |
|   |       +-- mock-tts/                 # Mock TTS provider
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               +-- provider/
|   |                   |-- mod.rs
|   |                   +-- fake.rs
|   |
|   |-- scripting/                        # Scripting/test interface
|   |   |-- Cargo.toml
|   |   +-- crates/
|   |       +-- scripting-api/            # Programmatic API
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               +-- runner/
|   |                   |-- mod.rs
|   |                   +-- scenario.rs   # Scenario runner
|   |
|   |-- spies/                            # Test spies for verification
|   |   |-- Cargo.toml
|   |   +-- crates/
|   |       +-- spy-api/                  # API call recording
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               +-- recorder/
|   |                   |-- mod.rs
|   |                   +-- calls.rs      # Call recording
|   |
|   |-- tests/                            # Integration tests
|   |   |-- Cargo.toml
|   |   +-- crates/
|   |       |-- test-scenarios/           # Scenario tests
|   |       |   |-- Cargo.toml
|   |       |   +-- src/
|   |       |       |-- lib.rs
|   |       |       |-- s001/
|   |       |       |   |-- mod.rs
|   |       |       |   +-- test.rs
|   |       |       +-- s002/
|   |       |           |-- mod.rs
|   |       |           +-- test.rs
|   |       |
|   |       +-- test-harness/             # Test utilities
|   |           |-- Cargo.toml
|   |           +-- src/
|   |               |-- lib.rs
|   |               +-- fixtures/
|   |                   |-- mod.rs
|   |                   +-- setup.rs
|   |
|   +-- web-ui/                           # Web UI component
|       |-- Cargo.toml
|       +-- crates/
|           |-- ui-components/            # Shared UI components
|           |   |-- Cargo.toml
|           |   +-- src/
|           |       |-- lib.rs
|           |       |-- input/
|           |       |   |-- mod.rs
|           |       |   +-- text.rs       # Text input component
|           |       +-- button/
|           |           |-- mod.rs
|           |           +-- submit.rs     # Submit button
|           |
|           |-- ui-lab/                   # Scenario harness
|           |   |-- Cargo.toml
|           |   +-- src/
|           |       |-- lib.rs
|           |       |-- main.rs           # WASM entry
|           |       |-- router/
|           |       |   |-- mod.rs
|           |       |   +-- scenario.rs   # Scenario routing
|           |       +-- scenarios/
|           |           |-- mod.rs
|           |           |-- s001.rs
|           |           +-- s002.rs
|           |
|           +-- ui-app/                   # Production UI
|               |-- Cargo.toml
|               +-- src/
|                   |-- lib.rs
|                   |-- main.rs
|                   +-- pages/
|                       |-- mod.rs
|                       +-- home.rs
|
|-- e2e/                                  # Playwright test suite
|   |-- package.json
|   |-- playwright.config.ts
|   +-- tests/
|       |-- s001_basic_tts.spec.ts
|       +-- s002_voice_select.spec.ts
|
|-- docs/
|   |-- architecture.md                   # This file
|   |-- prd.md
|   |-- design.md
|   |-- plan.md
|   |-- status.md
|   +-- scenarios.md
|
+-- scripts/
    |-- fmt.sh              # Run cargo fmt on all components
    |-- clippy.sh           # Run cargo clippy on all components
    |-- build.sh            # Build all components
    |-- test.sh             # Test all components
    |-- check.sh            # Run sw-checklist on all components
    |-- run_backend.sh      # Start backend server (port 1100)
    |-- run_ui_lab.sh       # Start UI lab (port 1101)
    +-- run_e2e.sh          # Run Playwright E2E tests
```

### Root Scripts

All root scripts iterate over components in `components/` directory:

```bash
# scripts/fmt.sh
#!/bin/bash
set -euo pipefail
for component in components/*/; do
    echo "=== Formatting $component ==="
    (cd "$component" && cargo fmt --all)
done

# scripts/clippy.sh
#!/bin/bash
set -euo pipefail
for component in components/*/; do
    echo "=== Clippy $component ==="
    (cd "$component" && cargo clippy --all-targets --all-features -- -D warnings)
done

# scripts/build.sh
#!/bin/bash
set -euo pipefail
for component in components/*/; do
    echo "=== Building $component ==="
    (cd "$component" && cargo build --all-targets)
done

# scripts/test.sh
#!/bin/bash
set -euo pipefail
for component in components/*/; do
    echo "=== Testing $component ==="
    (cd "$component" && cargo test --all-targets)
done

# scripts/check.sh
#!/bin/bash
set -euo pipefail
for component in components/*/; do
    echo "=== Checking $component ==="
    sw-checklist "$component"
done
```

### Component Summary

| Component   | Purpose                              | Crates                                    |
|-------------|--------------------------------------|-------------------------------------------|
| backend     | TTS proxy server                     | backend-api, backend-proxy, backend-tts   |
| cli         | Command-line interface               | cli-app, cli-commands                     |
| core        | Shared types and client              | core-types, core-client, core-error, core-scenarios |
| mocks       | Mock implementations for testing     | mock-backend, mock-tts                    |
| scripting   | Programmatic test interface          | scripting-api                             |
| spies       | Test verification                    | spy-api                                   |
| tests       | Integration tests                    | test-scenarios, test-harness              |
| web-ui      | Browser-based UI                     | ui-components, ui-lab, ui-app             |

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
