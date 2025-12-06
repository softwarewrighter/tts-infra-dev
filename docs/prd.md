# Product Requirements Document (PRD)

## Product Overview

**Product Name**: tts-infra-dev

**Vision**: A compartmentalized, testable Text-to-Speech (TTS) infrastructure built with Rust/WASM that enables reliable AI coding agent development through scenario-driven testing.

**Problem Statement**: CLI-based AI coding agents struggle with complex web UIs containing modes, tabs, and dropdowns. They waste significant time navigating to specific test configurations, lose context frequently, and make repeated errors due to:
- Large, unfocused contexts
- Complex navigation requirements
- Unreliable UI element selection
- Lack of deterministic test entry points

## Goals

### Primary Goals

1. **Three Interfaces, One API**: Every use case accessible via CLI, Scripting, and Web UI
2. **Compartmentalized Testing**: Separate test UIs for each use case, each demonstrating minimum inputs/controls
3. **Scripting-First Regression**: Scripting tests validate API call sequences; CLI and UI follow same patterns
4. **Deterministic UI Entry Points**: Pre-configured scenario routes that eliminate navigation complexity
5. **AI Agent Productivity**: Reduce context exhaustion and improve developer velocity

### Success Metrics

| Metric                          | Target                |
|---------------------------------|-----------------------|
| Test setup time                 | < 5 seconds           |
| Context window usage per task   | < 30% of limit        |
| AI agent task completion rate   | > 90%                 |
| Regression test coverage        | > 80%                 |
| E2E test reliability            | > 95% pass rate       |

## User Personas

### 1. AI Coding Agent

**Needs**:
- Small, focused file sets per task
- Clear, single-purpose entry points
- Stable selectors (data-testid)
- Explicit checklists and goals
- Fast feedback loops

**Pain Points**:
- Complex navigation sequences
- Dropdown/modal interactions
- Large context windows
- Ambiguous requirements
- Multi-step debugging

### 2. Human Developer

**Needs**:
- Efficient TTS API integration
- Multiple backend support
- Clean architecture
- Comprehensive test coverage
- Fast iteration cycles

**Pain Points**:
- Backend flakiness during development
- UI testing complexity
- Cross-platform compatibility
- API documentation gaps

## Functional Requirements

### FR1: Core API Library (core/)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR1.1 | Shared types for all interfaces                | P0       |
| FR1.2 | API client for backend communication           | P0       |
| FR1.3 | Scenario implementations (reusable logic)      | P0       |
| FR1.4 | Error types and handling                       | P0       |

### FR2: CLI Interface (cli/)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR2.1 | `tts health` - Check backend status            | P0       |
| FR2.2 | `tts synthesize "text"` - Basic TTS            | P0       |
| FR2.3 | `tts voices` - List available voices           | P0       |
| FR2.4 | `tts synthesize --voice=X --format=Y`          | P1       |
| FR2.5 | Output to file or stdout                       | P1       |
| FR2.6 | JSON output mode for scripting                 | P1       |

### FR3: TTS Backend API (backend/)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR3.1 | Health check endpoint (/health)                | P0       |
| FR3.2 | Text-to-speech synthesis (/synthesize)         | P0       |
| FR3.3 | Voice listing (/voices)                        | P0       |
| FR3.4 | Audio format selection (mp3, wav, ogg)         | P1       |
| FR3.5 | Rate limiting and usage tracking               | P1       |
| FR3.6 | Multi-provider proxy (OpenAI, ElevenLabs, etc) | P1       |

### FR4: Scripting Test Suite (scripting/)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR2.1 | reqwest-based scenario tests                   | P0       |
| FR2.2 | Scenario manifest with API sequences           | P0       |
| FR2.3 | Error case coverage                            | P0       |
| FR2.4 | Backend-down simulation                        | P1       |
| FR2.5 | Insta snapshot testing for JSON responses      | P2       |

### FR3: UI Lab (ui-lab)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR5.1 | Route-based scenario selection (?scenario=)   | P0       |
| FR5.2 | Pre-configured component state                 | P0       |
| FR5.3 | data-testid on all interactive elements        | P0       |
| FR5.4 | Scenario header display                        | P0       |
| FR5.5 | Optional ai_mode=1 for simplified DOM          | P1       |
| FR5.6 | Mock backend support                           | P1       |

### FR6: E2E Test Suite (e2e/)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR6.1 | Playwright tests via MCP                       | P0       |
| FR6.2 | One test file per scenario                     | P0       |
| FR6.3 | Direct URL access (no navigation)              | P0       |
| FR6.4 | data-testid based selectors                    | P0       |
| FR6.5 | Assertion on expected outcomes                 | P0       |

### FR7: Production UI (ui-app/)

| ID    | Requirement                                    | Priority |
|-------|------------------------------------------------|----------|
| FR7.1 | Text input for TTS                             | P0       |
| FR7.2 | Voice selection dropdown                       | P0       |
| FR7.3 | Audio playback of synthesized speech           | P0       |
| FR7.4 | Download audio file                            | P1       |
| FR7.5 | History of recent syntheses                    | P2       |
| FR7.6 | Unified UI combining all scenario UIs          | P2       |

## Non-Functional Requirements

### NFR1: Performance

| ID     | Requirement                                    | Target       |
|--------|------------------------------------------------|--------------|
| NFR1.1 | API response time (health)                     | < 50ms       |
| NFR1.2 | API response time (synthesis start)            | < 500ms      |
| NFR1.3 | UI lab load time                               | < 2s         |
| NFR1.4 | E2E test execution time (single)               | < 10s        |

### NFR2: Reliability

| ID     | Requirement                                    | Target       |
|--------|------------------------------------------------|--------------|
| NFR2.1 | API uptime                                     | 99%          |
| NFR2.2 | Test suite stability                           | > 95% green  |
| NFR2.3 | No flaky tests                                 | 0 allowed    |

### NFR3: AI Agent Compatibility

| ID     | Requirement                                    | Target                |
|--------|------------------------------------------------|-----------------------|
| NFR3.1 | Files per scenario                             | < 6                   |
| NFR3.2 | Lines per file                                 | < 300                 |
| NFR3.3 | Context needed per task                        | 1 scenario only       |
| NFR3.4 | Navigation steps to test state                 | 0 (direct URL)        |
| NFR3.5 | CLI command per scenario                       | 1 command             |

### NFR4: Maintainability

| ID     | Requirement                                    | Target       |
|--------|------------------------------------------------|--------------|
| NFR4.1 | Code coverage                                  | > 80%        |
| NFR4.2 | Documentation coverage                         | 100% public  |
| NFR4.3 | Clippy warnings                                | 0            |
| NFR4.4 | TODO comments per file                         | < 3          |

## Scenarios (Initial Set)

| ID   | Name             | CLI Command                              | Description                    |
|------|------------------|------------------------------------------|--------------------------------|
| S001 | Basic TTS        | `tts synthesize "Hello"`                 | Simple text to speech          |
| S002 | Voice Selection  | `tts synthesize "Hi" --voice=en-us`      | Select voice and synthesize    |
| S003 | Format Selection | `tts synthesize "Hi" --format=wav`       | Choose output format           |
| S004 | Long Text        | `tts synthesize @longtext.txt`           | Handle text > 1000 chars       |
| S005 | Error Handling   | `tts synthesize ""`                      | Invalid input shows error      |
| S006 | Backend Down     | `tts health` (when backend down)         | Graceful degradation           |
| S007 | Multi-Provider   | `tts synthesize "Hi" --provider=openai`  | Switch between providers       |
| S008 | Audio Playback   | (Web UI only)                            | Play audio in browser          |

## Constraints

1. **Technology**: Rust for backend and WASM UI (minimal JavaScript)
2. **Testing**: wasm-bindgen-test for Rust tests, Playwright for E2E
3. **Edition**: Rust 2024 edition
4. **AI Compatibility**: MCP-based Playwright integration required

## Out of Scope (v1)

- User authentication/accounts
- Batch processing
- Real-time streaming synthesis
- Mobile-specific UI
- Billing/payment integration

## Timeline

See docs/plan.md for implementation schedule.

## Risks and Mitigations

| Risk                            | Impact | Mitigation                               |
|---------------------------------|--------|------------------------------------------|
| TTS provider API changes        | High   | Abstraction layer for provider switching |
| WASM performance issues         | Medium | Benchmark early, optimize hot paths      |
| Playwright selector instability | Medium | Strict data-testid convention            |
| AI context exhaustion           | High   | Minimal file sets per scenario           |

## Appendix

### Three Interface Examples

**S001 Basic TTS**:

| Interface  | Usage                                           |
|------------|-------------------------------------------------|
| CLI        | `tts synthesize "Hello world"`                  |
| Scripting  | `client.synthesize("Hello world", None).await?` |
| Web UI     | Text input + Synthesize button                  |

All three call the same API sequence:
1. GET /health
2. POST /synthesize { text: "Hello world" }
3. Response: { audio_url: "...", duration_ms: ... }

**S002 Voice Selection**:

| Interface  | Usage                                                    |
|------------|----------------------------------------------------------|
| CLI        | `tts synthesize "Hello" --voice=en-us-male`              |
| Scripting  | `client.synthesize("Hello", Some("en-us-male")).await?`  |
| Web UI     | Text input + Voice dropdown + Synthesize button          |

API sequence:
1. GET /health
2. GET /voices
3. POST /synthesize { text: "...", voice_id: "..." }
