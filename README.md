# HALPI2 Workspace

HALPI2 is a Raspberry Pi Compute Module 5 carrier board designed for embedded marine and industrial applications. This workspace manages multiple independent repositories for convenient development.

## What is HALPI2?

HALPI2 provides a ruggedized Raspberry Pi platform with:

- **Robust power management** via RP2040 microcontroller with supercapacitor backup
- **Marine-grade interfaces**: CAN bus, RS-485, multiple I2C buses
- **Smart power control**: Automatic shutdown on power loss, watchdog functionality
- **Web-based monitoring**: Power status and system control via daemon
- **Industrial reliability**: Designed for 24/7 operation in harsh environments

**Use cases:**
- Marine electronics (NMEA 2000, instrument monitoring)
- Industrial automation and IoT gateways
- Remote monitoring and control systems
- Embedded computing with power resilience

## Repository Structure

This is a **workspace repository** containing multiple independent git repositories. Each can be checked out and worked on independently.

### Core System Components

- **HALPI2-hardware/** - Main carrier board PCB design (KiCad 9.0)
  - Repository: `git@github.com:hatlabs/HALPI2-hardware.git`
  - RP2040 power management, CM5 carrier, marine interfaces

- **HALPI2-firmware/** - RP2040 embedded firmware (Rust/Embassy)
  - Repository: `git@github.com:hatlabs/HALPI2-firmware.git`
  - State machine-based power management
  - I2C secondary interface for CM5 communication

- **HALPI2-rust-daemon/** - Power monitoring daemon (Rust)
  - Repository: `git@github.com:hatlabs/HALPI2-rust-daemon.git`
  - Runs on CM5 Linux, manages power events and watchdog
  - Provides `halpi` CLI and HTTP API

- **HALPI2-python-daemon/** - Power monitoring daemon (Python, archived)
  - Repository: `git@github.com:hatlabs/HALPI2-daemon.git`
  - Superseded by HALPI2-rust-daemon; kept as reference

- **halpi2/** - User documentation (mdBook)
  - Repository: `git@github.com:hatlabs/halpi2.git`
  - Published at https://docs.hatlabs.fi/halpi2

### Hardware Components

- **HALPI2-HDMI-FPC/** - Impedance-controlled HDMI flexible PCB (KiCad 9.0)
  - Repository: `git@github.com:hatlabs/HALPI2-HDMI-FPC.git`
  - 100Ω differential impedance for reliable HDMI

## Quick Start

### For Developers

```bash
# Clone all component repositories
./run repos:clone

# Update all repositories to latest
./run repos:pull-all-main

# Check status of all repositories
./run repos:status

# List managed repositories
./run repos:list
```

### Working with Individual Components

Each repository has its own build system and documentation:

```bash
# Firmware development
cd HALPI2-firmware
./run build && ./run flash:monitor

# Daemon development
cd HALPI2-rust-daemon
./run docker-build

# Documentation
cd halpi2
mdbook serve --open

# Hardware design
# Open KiCad projects directly
open HALPI2-hardware/HALPI2.kicad_pro
```

**Always read each repository's `CLAUDE.md`** for detailed development instructions.

### Adding Local Repositories

To manage additional repositories locally, create a `repos.*.sh` file in the workspace root (e.g., `repos.local.sh`):

```bash
# repos.local.sh (gitignored)
REPOS["my-private-repo"]="git@github.com:myorg/my-repo.git main"
```

All `repos.*.sh` files are sourced by the `./run` script and can add entries to the `REPOS` array.

## System Architecture

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

### Key Features

**Power Management State Machine** (core architectural component):
- Monitors input voltage and supercapacitor state
- Manages power rail sequencing (3.3V, 5V, 10V)
- Coordinates graceful shutdown on power loss
- Provides watchdog functionality
- See `HALPI2-firmware/state_machine.png` for visualization

**I2C Communication Protocol**:
- Commands: power control, watchdog, voltage thresholds, analog readings, shutdown, DFU
- Daemon → Firmware: Monitor and control
- Firmware → Hardware: GPIO and analog I/O

## Development Workflow

### Cross-Repository Changes

When changes span multiple repositories (e.g., new I2C command):

1. **Update firmware** with new I2C command
2. **Update daemon** to use new command
3. **Update documentation** with new feature
4. **Commit changes separately** in each repository
5. **Coordinate versions** if breaking changes

Example:
```bash
# Add new I2C command to firmware
cd HALPI2-firmware
# make changes
git commit -m "feat(i2c): add USB power cycling command"
git push

# Use new command in daemon
cd ../HALPI2-rust-daemon
# make changes
git commit -m "feat(usb): add power cycling support"
git push

# Document new feature
cd ../halpi2
# update docs
git commit -m "docs(features): add USB power cycling"
git push
```

### Git Workflow

Each repository uses **conventional commits**:

```
<type>(<scope>): <subject>

[optional body]
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`

## Resources

- **Product Page**: https://shop.hatlabs.fi/products/halpi2
- **Documentation**: https://docs.hatlabs.fi/halpi2
- **APT Repository**: https://apt.hatlabs.fi
- **Support**: GitHub issues in individual repositories

## License

- **Hardware**: CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S v2)
- **Software**: See individual repository licenses

## Contributing

Contributions are welcome! Each repository accepts pull requests independently. See individual repository documentation for specific guidelines.
