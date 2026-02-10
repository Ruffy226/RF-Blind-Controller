# CLAUDE.md - RF Blind Controller

This file provides guidance for AI assistants working on this repository.

## Project Overview

**RF-Blind-Controller** is a Radio Frequency (RF) based controller for motorized window blinds/shades. The project is in its initial stages — the repository has been created but does not yet contain source code or build infrastructure.

## Repository Status

- **State:** Empty / newly initialized
- **Remote:** Configured at `origin`
- **Default branch:** Not yet established (no commits on main/master)

## Intended Purpose

Based on the project name, this is expected to be an embedded systems project for controlling motorized blinds via RF signals. Typical components for such a project include:

- Microcontroller firmware (e.g., ESP8266/ESP32, Arduino, STM32)
- RF transceiver communication (e.g., 433MHz, 868MHz modules)
- Blind motor control logic (open, close, position)
- Possible wireless protocol implementation (e.g., Somfy RTS, custom protocol)

## Development Guidelines

### Getting Started

When this project gains source code, update this section with:

- Required hardware and development tools
- Build and flash instructions
- Dependencies and how to install them

### Code Conventions

- Use clear, descriptive names for functions and variables
- Comment non-obvious hardware-specific logic
- Keep ISR (interrupt service routines) minimal
- Separate hardware abstraction from application logic
- Document pin assignments and RF protocol details

### Git Workflow

- Use descriptive commit messages explaining *why* a change was made
- Keep commits focused on a single logical change
- Branch names should follow the pattern: `feature/<name>`, `fix/<name>`, `docs/<name>`

### File Organization (Recommended)

```
RF-Blind-Controller/
├── CLAUDE.md              # This file - AI assistant guidance
├── README.md              # Project documentation for users
├── src/                   # Source code
│   ├── main.*             # Entry point
│   ├── rf/                # RF communication modules
│   ├── motor/             # Blind motor control
│   └── config/            # Configuration and pin definitions
├── include/               # Header files (if C/C++)
├── lib/                   # Local libraries
├── test/                  # Unit and integration tests
├── docs/                  # Additional documentation
└── platformio.ini         # Build configuration (if using PlatformIO)
    OR Makefile            # Build configuration (if using Make)
    OR CMakeLists.txt      # Build configuration (if using CMake)
```

### Testing

- Add unit tests for protocol logic and state machines
- Hardware-dependent code should have mock interfaces for testing
- Test RF timing and signal generation where possible

## Notes for AI Assistants

- This repository is newly created. When adding initial code, confirm the target hardware platform and build system with the user before proceeding.
- If generating embedded code, be mindful of memory constraints, real-time timing requirements, and interrupt safety.
- Always read existing files before modifying them — do not assume file contents.
- Update this CLAUDE.md file as the project evolves to reflect the actual structure, build commands, and conventions in use.
