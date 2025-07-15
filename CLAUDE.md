# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a pure Rust implementation of the Android Debug Bridge (ADB) client protocol. The project consists of three main components:

- **adb_client**: Core library implementing ADB protocols for both server and direct device communication
- **adb_cli**: Command-line interface using the adb_client library
- **pyadb_client**: Python bindings for the adb_client library

## Development Commands

### Build Commands
```bash
# Build all workspace members
cargo build --release --all-features

# Build specific package
cargo build -p adb_client --release --all-features
cargo build -p adb_cli --release --all-features
cargo build -p pyadb_client --release --all-features
```

### Testing and Quality
```bash
# Run all tests
cargo test --verbose --all-features

# Run linting
cargo clippy --all-features

# Format code
cargo fmt --all --check

# Generate documentation
cargo doc --all-features --no-deps

# Run benchmarks
cargo bench
```

### Local Development
When working on a new release locally, the workspace uses path dependencies via `[patch.crates-io]` to reference the local `adb_client` crate.

## Architecture Overview

### Core Device Types
The library provides multiple ways to connect to Android devices:

1. **ADBServerDevice** (`server_device/`): Connects through ADB server (standard adb behavior)
2. **ADBUSBDevice** (`device/adb_usb_device.rs`): Direct USB connection to device
3. **ADBTcpDevice** (`device/adb_tcp_device.rs`): Direct TCP/IP connection to device  
4. **ADBEmulatorDevice** (`emulator_device/`): Specialized emulator communication

### Transport Layer
The transport layer (`transports/`) abstracts different connection types:
- **TCPServerTransport**: Communication with ADB server
- **USBTransport**: Direct USB device communication
- **TcpTransport**: Direct TCP device communication
- **TCPEmulatorTransport**: Emulator-specific transport

### Message Protocol
Device communication uses the ADB message protocol (`device/adb_transport_message.rs`):
- **ADBTransportMessage**: Core message structure
- **MessageCommand**: Command types (AUTH, CNXN, OPEN, etc.)
- Authentication handled via RSA keys (`device/models/adb_rsa_key.rs`)

### Command Implementation
Commands are organized by device type:
- **device/commands/**: Direct device commands (shell, push, pull, install, etc.)
- **server/commands/**: ADB server commands (devices, connect, disconnect, etc.)
- **server_device/commands/**: Server-mediated device commands
- **emulator_device/commands/**: Emulator-specific commands (SMS, rotate, etc.)

### Key Features
- **Pure Rust**: No shell command execution - implements ADB protocol directly
- **Multiple connection modes**: Server proxy, direct USB, direct TCP/IP
- **Advanced features**: Framebuffer capture, mDNS device discovery
- **Authentication**: RSA key-based device authentication
- **Cross-platform**: Windows, macOS, Linux support

### Error Handling
Centralized error handling via `error.rs` with `RustADBError` enum covering all failure modes.

### Constants and Models
- **constants.rs**: Protocol constants and magic numbers
- **models/**: Data structures for requests/responses across all components

## Testing Notes
- Tests require physical devices or emulators to be connected
- Use `cargo test --verbose --all-features` for comprehensive testing
- Benchmarks available for performance-critical operations like file push