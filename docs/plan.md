# Implementation Plan

## Overview

This document outlines the phased implementation plan for tts-infra-dev. Each phase builds on the previous, establishing a solid foundation before adding complexity.

## Phase 0: Project Foundation

### Objectives
- Set up Cargo workspace structure
- Establish development tooling
- Create initial documentation

### Tasks

- [ ] Create workspace Cargo.toml with member crates
- [ ] Set up core crate (shared types, client, scenarios)
- [ ] Set up cli crate with clap
- [ ] Set up backend crate with basic structure
- [ ] Set up scripting crate with test harness
- [ ] Configure clippy, rustfmt, and pre-commit checks
- [ ] Create docs/scenarios.md template
- [ ] Create docs/ports.md with port assignments (110x range)
- [ ] Add .gitignore for Rust/WASM artifacts

### Deliverables
- Compilable workspace with empty crates
- Three-interface structure established (core, cli, scripting)
- Development process documented
- CI-ready project structure

---

## Phase 1: Core API and Backend

### Objectives
- Implement core types and client
- Implement backend API endpoints
- Establish three-interface pattern with S001

### Tasks

**Core (core/)**
- [ ] Define SynthesizeRequest/Response types
- [ ] Implement Client with health() and synthesize() methods
- [ ] Create S001 scenario function
- [ ] Add error types

**Backend (backend/)**
- [ ] Implement GET /health endpoint (port 1100)
- [ ] Implement POST /synthesize with mock TTS
- [ ] Add basic error handling (ApiError type)
- [ ] Configure CORS for local development

**CLI (cli/)**
- [ ] Implement `tts health` command
- [ ] Implement `tts synthesize "text"` command
- [ ] Use core crate for logic

**Scripting (scripting/)**
- [ ] Create test harness with base_url configuration
- [ ] Implement S001 Basic TTS scenario test
- [ ] Validate same logic as CLI
- [ ] Document S001 in scenarios.md

### Deliverables
- Working /health and /synthesize endpoints
- CLI commands working: `tts health`, `tts synthesize`
- S001 scripting test passing
- Three-interface pattern established

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
