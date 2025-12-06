# tts-infra-dev

Text-to-Speech infrastructure built with Rust/WASM, designed for compartmentalized testing and AI coding agent productivity.

## Overview

This project implements a TTS (Text-to-Speech) system with three interfaces sharing a common core:

- **CLI** - Command-line interface for power users and automation
- **Scripting** - Programmatic API for tests to drive correct call sequences
- **Web UI** - Browser-based interface for interactive use

## Quick Start

```bash
# Build all components
./scripts/build.sh

# Run tests
./scripts/test.sh

# Start backend (port 1100)
./scripts/run_backend.sh

# Start UI Lab (port 1101)
./scripts/run_ui_lab.sh
```

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](docs/architecture.md) | System architecture, component layout, physical structure |
| [PRD](docs/prd.md) | Product requirements, scenarios, success metrics |
| [Design](docs/design.md) | Design decisions with rationale and trade-offs |
| [Plan](docs/plan.md) | Phased implementation plan with task breakdowns |
| [Status](docs/status.md) | Current progress, scenario matrix, test results |
| [Process](docs/process.md) | Development workflow, TDD, pre-commit gates |
| [Tools](docs/tools.md) | Development tooling documentation |
| [AI Instructions](docs/ai_agent_instructions.md) | Guidelines for AI coding agents |

## Project Structure

```
tts-infra-dev/
+-- components/
|   +-- backend/          # TTS proxy server
|   +-- cli/              # Command-line interface
|   +-- core/             # Shared types and client
|   +-- mocks/            # Mock implementations
|   +-- scripting/        # Test interface
|   +-- spies/            # Test verification
|   +-- tests/            # Integration tests
|   +-- web-ui/           # Browser UI
+-- e2e/                  # Playwright tests
+-- docs/                 # Documentation
+-- scripts/              # Build and run scripts
```

## Component Layout

Each component follows this structure:

```
components/<name>/
+-- Cargo.toml            # Workspace manifest
+-- crates/
    +-- <crate-name>/
        +-- Cargo.toml    # Crate manifest (edition = "2024")
        +-- src/
            +-- lib.rs    # Re-exports only
            +-- <module>/
                +-- mod.rs    # Re-exports only
                +-- <file>.rs # Implementation
```

## Code Quality

This project enforces strict limits via `sw-checklist`:

| Metric              | Warn  | Fail  |
|---------------------|-------|-------|
| Lines per function  | >25   | >50   |
| Functions per module| >4    | >7    |
| Modules per crate   | >4    | >7    |

Additional rules:
- No functions in `lib.rs` or `mod.rs` (re-exports only)
- All crates use Rust Edition 2024
- Zero clippy warnings allowed

## Scripts

| Script | Purpose |
|--------|---------|
| `scripts/fmt.sh` | Format all components |
| `scripts/clippy.sh` | Lint all components |
| `scripts/build.sh` | Build all components |
| `scripts/test.sh` | Test all components |
| `scripts/check.sh` | Run sw-checklist on all components |
| `scripts/run_backend.sh` | Start backend server (port 1100) |
| `scripts/run_ui_lab.sh` | Start UI lab (port 1101) |
| `scripts/run_e2e.sh` | Run Playwright E2E tests |

## Port Assignments

| Service       | Port |
|---------------|------|
| Backend API   | 1100 |
| UI Lab        | 1101 |
| UI App        | 1102 |
| Mock Backend  | 1109 |

## Scenarios

| ID   | Name             | CLI Command                     | Description              |
|------|------------------|---------------------------------|--------------------------|
| S001 | Basic TTS        | `tts synthesize "Hello"`        | Simple text to speech    |
| S002 | Voice Selection  | `tts synthesize "Hi" --voice=X` | Select voice             |
| S003 | Format Selection | `tts synthesize "Hi" --format=wav` | Choose output format  |
| S004 | Long Text        | `tts synthesize @file.txt`      | Handle long text         |
| S005 | Error Handling   | `tts synthesize ""`             | Invalid input handling   |
| S006 | Backend Down     | `tts health`                    | Graceful degradation     |
| S007 | Multi-Provider   | `tts synthesize --provider=X`   | Provider switching       |
| S008 | Audio Playback   | (Web UI only)                   | Browser audio playback   |

## License

MIT License - see [LICENSE](LICENSE) for details.

## Copyright

Copyright (c) 2024 Software Wrighter LLC
