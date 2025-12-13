# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Swift Protobuf is Apple's official Swift implementation of Google's Protocol Buffers. It provides:
- **protoc-gen-swift**: A plugin for Google's `protoc` compiler that generates Swift code from `.proto` files
- **SwiftProtobuf**: A runtime library for working with generated protobuf types
- **SwiftProtobufPluginLibrary**: A library for building custom protoc plugins
- **SwiftProtobufPlugin**: An SPM build tool plugin that integrates protoc code generation

## Build Commands

```bash
# Build all targets (debug)
swift build

# Build for release
swift build -c release

# Run tests (runtime + plugin library tests)
swift test

# Run all tests including plugin output verification
make test

# Run only runtime library tests
make test-runtime

# Run plugin output verification (compares generated output against Reference/)
make test-plugin

# Run SPM plugin tests
make test-spm-plugin

# Run conformance tests (requires building conformance runner from protobuf checkout)
make test-conformance

# Run compile tests (multi-module and internal imports scenarios)
make compile-tests
```

## Regenerating Proto Files

After making changes to protoc-gen-swift code generation:

```bash
# 1. Build the plugin
make build

# 2. Regenerate all Swift files from protos
make regenerate

# 3. Rebuild to compile with regenerated files
make build

# 4. Run tests to verify
make test

# 5. Update reference files (MANUALLY verify diff before committing)
make reference
```

Specific regeneration targets:
- `make regenerate-library-protos` - SwiftProtobuf library protos
- `make regenerate-plugin-protos` - Plugin library protos
- `make regenerate-test-protos` - Test protos
- `make regenerate-conformance-protos` - Conformance test protos

## Architecture

### Source Structure

- **Sources/SwiftProtobuf/**: Runtime library - serialization, deserialization, type support
- **Sources/SwiftProtobufPluginLibrary/**: Library for building protoc plugins, descriptor APIs
- **Sources/protoc-gen-swift/**: The Swift code generator plugin
- **Sources/protobuf/**: Embedded protoc compiler (C++, built via SPM)
- **Sources/Conformance/**: Conformance test runner

### Generated Code Patterns

The plugin generates different storage patterns based on message complexity:

1. **Simple structs**: Messages with only basic fields use direct property storage
2. **Storage class**: Messages with message-valued fields OR >16 fields use a private `_StorageClass` with copy-on-write semantics

Key generated components:
- `_protobuf_nameMap`: Field number ↔ name mapping for JSON/text format
- `traverse<V: Visitor>()`: Visitor pattern for serialization
- `decodeMessage<D: Decoder>()`: Decoder-driven deserialization

### Testing

- **SwiftProtobufTests/**: Runtime library unit and functional tests
- **SwiftProtobufPluginLibraryTests/**: Plugin library tests
- **protoc-gen-swiftTests/**: Code generator tests
- **Reference/**: Expected plugin output for regression testing
- **CompileTests/**: Multi-module and internal imports compile verification

## Code Style

- **Indentation**: 2 spaces (no tabs)
- **Line length**: 80 characters
- **File naming**: One type per file matching type name, or plural noun for grouped types
- **Extensions**: Name files `Type+Protocol.swift` for protocol conformance extensions

## Key Makefile Variables

Override these when invoking make:
- `PROTOC`: Path to protoc (default: `.build/debug/protoc`)
- `GOOGLE_PROTOBUF_CHECKOUT`: Path to protobuf repo (default: `Sources/protobuf/protobuf`)
- `SWIFT_BUILD_TEST_HOOK`: Extra args for swift build/test

## Swift Version

The package requires Swift 6.0+ (Xcode 16.0+) and uses Swift language mode 6 with strict concurrency.
