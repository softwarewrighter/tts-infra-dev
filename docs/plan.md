# Implementation Plan

## Overview

This document outlines the phased implementation plan for tts-infra-dev. Each phase builds on the previous, establishing a solid foundation before adding complexity.

## Phase 0: Project Foundation

### Objectives
- Set up component-based directory structure
- Create root scripts for build/test/lint
- Establish code organization patterns

### Tasks

**Directory Structure**
- [ ] Create components/ directory
- [ ] Create components/core/ with Cargo.toml workspace
- [ ] Create components/cli/ with Cargo.toml workspace
- [ ] Create components/backend/ with Cargo.toml workspace
- [ ] Create components/scripting/ with Cargo.toml workspace
- [ ] Create components/mocks/ with Cargo.toml workspace
- [ ] Create components/tests/ with Cargo.toml workspace

**Root Scripts**
- [ ] Create scripts/fmt.sh (format all components)
- [ ] Create scripts/clippy.sh (lint all components)
- [ ] Create scripts/build.sh (build all components)
- [ ] Create scripts/test.sh (test all components)
- [ ] Create scripts/check.sh (sw-checklist all components)

**Core Component Crates**
- [ ] Create components/core/crates/core-types/
- [ ] Create components/core/crates/core-error/
- [ ] Create components/core/crates/core-client/
- [ ] Create components/core/crates/core-scenarios/

**Documentation**
- [ ] Create docs/scenarios.md template
- [ ] Update .gitignore for multi-component structure

### Deliverables
- All component workspaces compile (empty)
- Root scripts work across all components
- sw-checklist passes on all components
- Component structure matches architecture.md

---

## Phase 1: Core Types and Client

### Objectives
- Implement core-types crate with request/response types
- Implement core-error crate with error types
- Implement core-client crate with API client
- Follow sw-checklist limits strictly

### Tasks

**core-types crate** (components/core/crates/core-types/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/request/mod.rs, src/request/synthesize.rs
- [ ] Create src/response/mod.rs, src/response/synthesize.rs, src/response/voice.rs
- [ ] Verify: <=4 modules, <=4 functions per module, <=25 LOC per function

**core-error crate** (components/core/crates/core-error/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/error/mod.rs, src/error/api.rs, src/error/client.rs
- [ ] Implement Error trait, Display, From conversions

**core-client crate** (components/core/crates/core-client/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/client/mod.rs, src/client/builder.rs, src/client/methods.rs
- [ ] Create src/config/mod.rs, src/config/settings.rs
- [ ] Implement health(), synthesize(), voices() methods

**core-scenarios crate** (components/core/crates/core-scenarios/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/s001/mod.rs, src/s001/basic_tts.rs
- [ ] Implement S001 scenario using core-client

**Validation**
- [ ] Run sw-checklist on components/core/
- [ ] Run clippy with -D warnings
- [ ] All tests pass

### Deliverables
- core-types, core-error, core-client, core-scenarios crates complete
- All crates pass sw-checklist (no warnings)
- S001 scenario implemented in core-scenarios

---

## Phase 1b: Backend API

### Objectives
- Implement backend component with HTTP endpoints
- Create mock TTS provider for testing

### Tasks

**backend-api crate** (components/backend/crates/backend-api/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/health/mod.rs, src/health/handler.rs
- [ ] Create src/synthesize/mod.rs, src/synthesize/handler.rs, src/synthesize/validation.rs
- [ ] Create src/voices/mod.rs, src/voices/handler.rs

**backend-tts crate** (components/backend/crates/backend-tts/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/provider/mod.rs, src/provider/trait_def.rs
- [ ] Create src/mock/mod.rs, src/mock/provider.rs (mock implementation)

**Backend Binary**
- [ ] Create backend-server crate with main.rs
- [ ] Wire up Axum routes to handlers
- [ ] Start on port 1100

**Validation**
- [ ] Run sw-checklist on components/backend/
- [ ] Backend starts and responds to /health

### Deliverables
- Backend running on port 1100
- GET /health, POST /synthesize, GET /voices working
- Mock TTS provider returns test data

---

## Phase 1c: CLI and Scripting

### Objectives
- Implement CLI component
- Implement scripting component for tests
- Verify three-interface pattern with S001

### Tasks

**cli-commands crate** (components/cli/crates/cli-commands/)
- [ ] Create src/lib.rs (re-exports only)
- [ ] Create src/health/mod.rs, src/health/run.rs
- [ ] Create src/synthesize/mod.rs, src/synthesize/run.rs, src/synthesize/args.rs
- [ ] Create src/voices/mod.rs, src/voices/run.rs

**cli-app crate** (components/cli/crates/cli-app/)
- [ ] Create src/main.rs (minimal, uses cli-commands)
- [ ] Configure clap with subcommands
- [ ] Verify --help and --version output

**scripting-api crate** (components/scripting/crates/scripting-api/)
- [ ] Create src/lib.rs (re-exports core-client and core-scenarios)
- [ ] Create src/runner/mod.rs, src/runner/scenario.rs

**test-scenarios crate** (components/tests/crates/test-scenarios/)
- [ ] Create src/lib.rs
- [ ] Create src/s001/mod.rs, src/s001/test.rs
- [ ] Write integration test for S001

**Validation**
- [ ] `tts health` command works
- [ ] `tts synthesize "Hello"` command works
- [ ] S001 test passes via scripting
- [ ] All components pass sw-checklist

### Deliverables
- CLI binary with health, synthesize, voices commands
- S001 test passing via scripting
- Three-interface pattern validated

---

## Phase 2: Voice Selection

### Objectives
- Add voice listing and selection
- Create S002 scenario across all interfaces
- Expand test coverage

### Tasks

**Core (core/)**
- [ ] Define Voice type
- [ ] Add voices() method to Client
- [ ] Update synthesize() for voice_id parameter
- [ ] Create S002 scenario function

**Backend**
- [ ] Implement GET /voices endpoint
- [ ] Add voice_id parameter to /synthesize
- [ ] Return available voices (mock data initially)

**CLI**
- [ ] Implement `tts voices` command
- [ ] Add `--voice` flag to `tts synthesize`

**Scripting**
- [ ] Implement S002 Voice Selection scenario test
- [ ] Test /voices endpoint
- [ ] Test synthesize with voice_id
- [ ] Document S002 in scenarios.md

### Deliverables
- Voice selection working across CLI, Scripting
- S001 and S002 tests passing

---

## Phase 3: UI Lab Foundation

### Objectives
- Create ui-lab crate with scenario routing
- Implement S001 and S002 UI scenarios
- Establish UI component patterns

### Tasks

**UI Lab (ui-lab/)**
- [ ] Set up Yew application with wasm-pack
- [ ] Use core crate for API logic (via wasm-bindgen)
- [ ] Create LabRoot component with scenario routing
- [ ] Implement query string parsing (?scenario=)
- [ ] Create scenario header component
- [ ] Implement S001 Basic TTS UI scenario
- [ ] Implement S002 Voice Selection UI scenario
- [ ] Add data-testid to all interactive elements
- [ ] Add loading and error state handling

**Build Scripts**
- [ ] Create scripts/run_ui_lab.sh (port 1101)
- [ ] Configure wasm-pack for development builds
- [ ] Set up static file serving

### Deliverables
- UI Lab running at localhost:1101
- S001 and S002 accessible via URL
- UI uses same core logic as CLI
- All elements have data-testid attributes

---

## Phase 4: E2E Testing Infrastructure

### Objectives
- Set up Playwright test suite
- Create E2E tests for S001 and S002
- Validate MCP integration

### Tasks

**E2E Tests (e2e/)**
- [ ] Initialize npm project with Playwright
- [ ] Configure playwright.config.ts for localhost:1101
- [ ] Create S001 Playwright test
- [ ] Create S002 Playwright test
- [ ] Verify tests work via MCP (Claude Code)
- [ ] Document Playwright patterns in ai_playbook.md

**Scripts**
- [ ] Create scripts/run_e2e.sh
- [ ] Add combined test runner script

### Deliverables
- Playwright tests passing for S001 and S002
- All three interfaces (CLI, Scripting, UI) validated
- MCP integration verified
- E2E testing patterns documented

---

## Phase 5: Error Scenarios

### Objectives
- Add error handling scenarios
- Test graceful degradation
- Improve robustness

### Tasks

**Scenarios**
- [ ] S005: Error Handling (invalid input)
- [ ] S006: Backend Down (connection failure)

**Backend**
- [ ] Add input validation for /synthesize
- [ ] Return appropriate error codes
- [ ] Add request logging

**UI Lab**
- [ ] Add error message display component
- [ ] Handle API errors gracefully
- [ ] Show loading states

**Tests**
- [ ] API tests for error cases
- [ ] E2E tests for error display

### Deliverables
- Error scenarios working end-to-end
- Robust error handling in place

---

## Phase 6: TTS Provider Integration

### Objectives
- Integrate real TTS providers
- Implement proxy layer
- Add provider switching

### Tasks

**Backend**
- [ ] Define TtsProvider trait
- [ ] Implement OpenAI TTS provider
- [ ] Implement provider configuration (env vars)
- [ ] Add provider selection logic
- [ ] Implement S007 Multi-Provider scenario

**Configuration**
- [ ] Create config.toml for provider settings
- [ ] Add environment variable handling
- [ ] Document provider setup

**Tests**
- [ ] Add provider-specific tests (with mocks)
- [ ] Test provider fallback logic

### Deliverables
- Real TTS synthesis working
- Provider switching implemented

---

## Phase 7: Audio Playback

### Objectives
- Add audio playback in UI
- Implement download functionality
- Create S008 scenario

### Tasks

**UI Lab**
- [ ] Create AudioPlayer component
- [ ] Handle audio URL playback
- [ ] Add download button
- [ ] Implement S008 Audio Playback scenario

**Tests**
- [ ] E2E test for audio playback visibility
- [ ] Test download functionality

### Deliverables
- Audio playback working in browser
- S008 scenario complete

---

## Phase 8: AI Mode and Polish

### Objectives
- Implement AI mode flag
- Simplify DOM for AI agents
- Add format selection (S003)

### Tasks

**UI Lab**
- [ ] Add AiModeContext
- [ ] Implement simplified components for ai_mode=1
- [ ] Create S003 Format Selection scenario
- [ ] Add S004 Long Text scenario

**Documentation**
- [ ] Complete ai_playbook.md
- [ ] Update all scenario documentation
- [ ] Review and update architecture docs

### Deliverables
- AI mode working
- All initial scenarios complete
- Documentation finalized

---

## Phase 9: Production UI

### Objectives
- Create production-quality UI
- Implement full user experience
- Polish and optimize

### Tasks

**UI App (ui-app/)**
- [ ] Create production Yew application
- [ ] Implement styled components
- [ ] Add history feature
- [ ] Implement responsive design
- [ ] Optimize WASM bundle size

**Testing**
- [ ] Add wasm-bindgen-test unit tests
- [ ] Verify E2E tests still pass
- [ ] Performance testing

### Deliverables
- Production-ready UI
- Full feature set implemented

---

## Phase 10: CI/CD and Release

### Objectives
- Set up continuous integration
- Prepare for deployment
- Create release artifacts

### Tasks

**CI/CD**
- [ ] Create GitHub Actions workflow
- [ ] Run all tests in CI
- [ ] Build release artifacts
- [ ] Add badge to README

**Documentation**
- [ ] Update README with usage instructions
- [ ] Create CHANGELOG.md
- [ ] Add deployment guide

### Deliverables
- CI pipeline running
- Release-ready project

---

## Scenario Summary

| Phase | Scenarios Added                        | Interfaces        |
|-------|----------------------------------------|-------------------|
| 1     | S001 Basic TTS                         | CLI, Scripting    |
| 2     | S002 Voice Selection                   | CLI, Scripting    |
| 3     | (UI implementation of S001, S002)      | Web UI            |
| 5     | S005 Error Handling, S006 Backend Down | All three         |
| 6     | S007 Multi-Provider                    | CLI, Scripting    |
| 7     | S008 Audio Playback                    | Web UI            |
| 8     | S003 Format Selection, S004 Long Text  | All three         |

## Dependencies

```
Phase 0 (Foundation)
    |
    v
Phase 1 (Backend Core) --> Phase 2 (Voices)
    |                           |
    v                           v
Phase 3 (UI Lab) <--------------+
    |
    v
Phase 4 (E2E Tests)
    |
    v
Phase 5 (Error Scenarios)
    |
    v
Phase 6 (Providers) --> Phase 7 (Audio)
    |                       |
    v                       v
Phase 8 (AI Mode) <---------+
    |
    v
Phase 9 (Production UI)
    |
    v
Phase 10 (CI/CD)
```

## Tracking Progress

Progress is tracked in docs/status.md with the following format:

```markdown
## Current Phase: X

### Completed
- [x] Task description

### In Progress
- [ ] Task description (assigned to: ...)

### Blocked
- [ ] Task description (blocked by: ...)
```

## Risk Mitigation

| Risk                     | Mitigation                              |
|--------------------------|-----------------------------------------|
| Provider API limitations | Start with mocks, add real providers later |
| WASM performance         | Benchmark early in Phase 3             |
| E2E flakiness            | Strict data-testid convention          |
| Scope creep              | Stick to scenario-first approach       |
