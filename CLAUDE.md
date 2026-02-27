# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## HALPI2 Workspace

HALPI2 - Raspberry Pi Compute Module 5 carrier board for embedded marine and industrial applications.

This workspace manages multiple independent repositories for convenience. Each repository must work independently and can have no directory-level cross-dependencies.

## Local Environment Setup

**For additional project context, see @CLAUDE.private.md and @CLAUDE.local.md** (optional, not included in this repository).

## Contributing Guidelines

### Code Standards

- Follow YAGNI, SOLID, DRY, and KISS principles
- Write self-documenting code; comments explain "why", not "what"
- Keep functions small and focused on a single responsibility
- Prefer composition over inheritance
- No magic numbers; use named constants
- Use strict type checking; avoid `any` or equivalent escape hatches
- Validate external inputs at system boundaries
- All new code requires tests — test behavior, not implementation details
- Documentation describes current state, not development history

### Git Workflow

- Branch from main for new work; never push directly to main
- Branch naming: `<type>/<description>` where type = feat|fix|docs|chore|refactor|test
- Conventional commits: `<type>(<scope>): <subject>` — 50 char subject max, imperative mood
- Atomic commits: one logical change per commit
- Clean up history via rebase before creating a PR
- Use rebase to update branches with upstream changes, never merge commits

### Pull Requests

- One logical change per PR; refactoring and behavior changes belong in separate PRs
- Descriptive titles suitable for release notes (under 70 characters)
- Descriptions explain motivation (why) and approach (how), not mechanics (what)
- Reference issues with `closes`, `fixes`, or `resolves` (e.g., "closes #18")
- All CI checks must pass before merging — no exceptions
- Use merge commits (not squash) to preserve commit history

## Independent Repositories

### Core System Components

**HALPI2-firmware/** - RP2040 embedded firmware (Rust/Embassy)
- Repository: `git@github.com:hatlabs/HALPI2-firmware.git`
- Hierarchical state machine for power management
- I2C secondary interface for CM5 communication
- Has its own `CLAUDE.md` and `./run` script

**HALPI2-rust-daemon/** - Power monitoring daemon (Rust)
- Repository: `git@github.com:hatlabs/HALPI2-rust-daemon.git`
- Runs on CM5 Linux, communicates with RP2040 via I2C
- Provides CLI (`halpi`) and daemon (`halpid`) with watchdog functionality
- Build in dev container: `cd HALPI2-rust-daemon && ./run docker-build`
- Has its own `CLAUDE.md` and `./run` script with Docker build commands

**HALPI2-python-daemon/** - Power monitoring daemon (Python, archived)
- Repository: `git@github.com:hatlabs/HALPI2-daemon.git`
- Superseded by HALPI2-rust-daemon; kept as reference

**HALPI2-hardware/** - Main carrier board PCB design (KiCad 9.0)
- RP2040-based power management and peripheral control
- Raspberry Pi CM5 carrier with marine-grade features
- Custom power supply with supercapacitor backup

**halpi2/** - User documentation (mdBook)
- Built with `mdbook build` / `mdbook serve --open`
- Deployed to GitHub Pages
- Has its own `CLAUDE.md`

### Additional Hardware Components

**HALPI2-HDMI-FPC/** - Impedance-controlled flexible PCB for HDMI (KiCad 9.0)
- 100Ω differential impedance for reliable HDMI connectivity
- Has its own `CLAUDE.md` and `./prepare_assembly.sh` script

**HALPI2-enclosure/** - Mechanical enclosure design files

**FFC-HDMI/** - Legacy flat flexible cable (superseded by HALPI2-HDMI-FPC)

## System Architecture

### Hardware + Firmware + Software Integration

```
┌─────────────────────────────────────────────────────┐
│  Raspberry Pi CM5 (Linux)                           │
│  ┌──────────────────────────────────────────┐      │
│  │  halpid (daemon)                         │      │
│  │  - Monitors power state                  │      │
│  │  - Feeds watchdog                        │      │
│  │  - Orchestrates shutdown                 │      │
│  └────────────────┬─────────────────────────┘      │
└───────────────────┼──────────────────────────────────┘
                    │ I2C (bus 1, addr 0x6d)
┌───────────────────┼──────────────────────────────────┐
│  RP2040 MCU       │                                  │
│  ┌────────────────┴─────────────────────────┐       │
│  │  HALPI2-firmware (Rust/Embassy)          │       │
│  │  - State machine (power management)      │       │
│  │  - GPIO control (power rails, USB, LEDs) │       │
│  │  - Analog monitoring (VIN, VSCAP, IIN)   │       │
│  │  - I2C secondary device                  │       │
│  └──────────────────────────────────────────┘       │
└──────────────────────────────────────────────────────┘
```

### Power Management State Machine

The **core architectural component** is the firmware state machine that coordinates power transitions:

- **Key States**: PowerOff, OffCharging, SystemStartup, OperationalSolo/CoOp, BlackoutSolo/CoOp, EnteringStandby, Standby, HostUnresponsive, PoweredDownBlackout/Manual
- **Triggers**: VIN voltage (>9.0V), supercap voltage (8.0V power-on, 5.5V power-off), CM5 status (3.3V rail), watchdog timeouts
- **Coordination**: RP2040 firmware manages hardware state; daemon handles graceful Linux shutdown

State machine visualization: `HALPI2-firmware/state_machine.png`

### I2C Communication Protocol

The firmware exposes an I2C secondary interface (address 0x6d) with commands:
- **0x10**: Power control
- **0x12**: Watchdog feed
- **0x13-0x14**: Voltage threshold configuration
- **0x20-0x23**: Analog readings (VIN, VSCAP, IIN)
- **0x30-0x31**: Shutdown commands
- **0x40-0x45**: DFU firmware update operations

The daemon uses these commands to monitor and control the system.

## Quick Start

```bash
# Clone all component repositories
./run repos:clone

# Update all repositories to latest
./run repos:pull-all-main

# Check status of all repositories
./run repos:status

# Work in a specific repository
cd HALPI2-firmware
./run build && ./run flash:monitor
```

**Each repository has its own CLAUDE.md** - read the appropriate one for detailed context.

## Building HALPI2-rust-daemon on macOS

The Rust daemon requires a Linux environment for building and testing. It has Docker-based build commands in its `./run` script:

```bash
cd HALPI2-rust-daemon

# Build in dev container (debug build)
./run docker-build

# Build in dev container (release build)
./run docker-build --release

# Cross-compile for ARM64 in dev container
./run docker-cross-build --release

# Build Debian package for ARM64
./run docker-build-deb
```

See `HALPI2-rust-daemon/CLAUDE.md` for detailed instructions.

## Repository Management

This workspace provides convenience commands for managing all repositories:

```bash
# Clone all missing repositories
./run repos:clone

# Update all repositories to latest main branches
./run repos:pull-all-main

# Check status of all repositories
./run repos:status

# List all managed repositories
./run repos:list
```

Each repository is managed independently. The HALPI2 workspace tracks only shared documentation and convenience scripts.

Additional repositories can be added via `repos.*.sh` files (gitignored) — see README.md for details.

## Working with the Workspace

### Per-Repository Operations

Each repository has its own:
- Git history and remote
- Build system and dependencies
- Version number
- `CLAUDE.md` file with detailed instructions
- `./run` script (where applicable)

**Always `cd` into the specific repository directory first** before running commands.

### Cross-Repository Workflows

When changes span multiple repositories (e.g., firmware + daemon + docs):

1. **Check each repository's CLAUDE.md** for specific build/test commands
2. **Make changes in each repo independently** with separate commits
3. **Coordinate versions** if interfaces change (especially I2C protocol)

Example firmware update workflow:
```bash
# Update firmware
cd HALPI2-firmware
./run build && ./run flash:monitor

# Update daemon if I2C commands changed
cd ../HALPI2-rust-daemon
./run check

# Update documentation
cd ../halpi2
mdbook serve --open
```

### Common Development Commands

Since each repo is independent, refer to the per-repository CLAUDE.md:

- **Firmware**: `cd HALPI2-firmware && ./run help`
- **Daemon**: `cd HALPI2-rust-daemon && ./run help`
- **Docs**: `cd halpi2 && mdbook serve --open`
- **Hardware**: Open KiCad projects directly

## Key Architectural Patterns

### Firmware (HALPI2-firmware/)

- **Embassy async framework**: All tasks run concurrently
- **State machine**: Uses `statig` crate for hierarchical states
- **Flash storage**: Sequential-storage for config persistence
- **GPIO**: 30+ pins controlling power, USB, LEDs, I2C
- **No traditional tests**: Hardware-specific embedded firmware

### Daemon (HALPI2-rust-daemon/)

- **Async runtime**: Tokio-based async I/O
- **I2C communication**: Linux I2C device access
- **Unix socket API**: HTTP server on `/run/halpid/halpid.sock`
- **State monitoring**: Polls firmware for blackout/voltage status
- **Watchdog**: Feeds firmware watchdog every ~5 seconds

### Hardware (HALPI2-hardware/)

- **KiCad 9.0**: All PCB designs
- **Hierarchical schematics**: Organized by functional block
- **Python tooling**: Jupyter notebooks for calculations
- **Panelization**: KiKit for manufacturing panel generation

## Environment Setup

The development environment may be macOS or Linux-based. Code and scaffolding
should work on either platform.

**Per-repository dependencies:**
- Firmware: Rust + thumbv6m-none-eabi target
- Daemon: Rust (builds in Docker dev container)
- Docs: Rust + `mdbook`
- Hardware: KiCad 9.0 + KiKit + Python

## Resources

- Product page: https://shop.hatlabs.fi/products/halpi2
- Documentation: https://docs.hatlabs.fi/halpi2
- APT repository: https://apt.hatlabs.fi
- Licenses: CERN-OHL-S v2 (hardware), per-project (software)
