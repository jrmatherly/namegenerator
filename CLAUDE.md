# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**namegenerator** is a lightweight Go library for generating random, human-readable names in the format `adjective-noun` (e.g., "silent-moon", "ancient-river"). It's designed for naming temporary resources like Docker containers, Kubernetes pods, test data, or temporary files.

- **Module**: `github.com/jrmatherly/namegenerator`
- **Go Version**: 1.23.2
- **Dependencies**: Zero external dependencies (uses only Go standard library)
- **Size**: ~135 lines across 3 source files
- **License**: Apache 2.0

## Essential Commands

### Testing

```bash
# Run all tests
go test

# Run specific test
go test -run TestNameGenerator_Generate

# Run with verbose output
go test -v

# Run with coverage report
go test -cover
go test -coverprofile=coverage.out
go tool cover -html=coverage.out

# Run with race detector (important - NameGenerator is NOT thread-safe)
go test -race
```

### Code Quality

```bash
# Format code (always run before committing)
go fmt ./...

# Static analysis
go vet ./...

# Tidy dependencies
go mod tidy
```

### Pre-Commit Workflow

```bash
# Run in this order before every commit:
go fmt ./...
go vet ./...
go test ./...
go mod tidy
```

## Architecture

### Core Design Pattern: Interface-First with Seeded PRNG

The library uses a **factory function pattern** that returns an interface, hiding the concrete implementation:

```
NewNameGenerator(seed) → Generator interface → NameGenerator struct
```

**Key architectural decisions:**

1. **Interface Return Type**: `NewNameGenerator()` returns the `Generator` interface (not `*NameGenerator`), allowing for future alternative implementations without breaking API.

2. **Seeded Randomness**: Each `NameGenerator` instance contains its own `*rand.Rand` initialized with a user-provided seed. This ensures:
   - Deterministic sequences when using the same seed
   - Independent random streams for different instances
   - Reproducible behavior for testing

3. **NOT Thread-Safe**: `NameGenerator` instances are **not safe for concurrent use** because `math/rand.Rand` is not thread-safe. For concurrent usage:
   - Create separate generator instances per goroutine
   - Or wrap calls in a mutex
   - Never share a single generator across goroutines

### Component Breakdown

**generator.go** (~55 lines)

- `Generator` interface - Single method: `Generate() string`
- `NameGenerator` struct - Contains `random *rand.Rand` field
- `NewNameGenerator(seed int64) Generator` - Factory function
- `Generate() string` method - Selects random adjective + noun, formats as "adjective-noun"

**data.go** (~43 lines)

- `ADJECTIVES []string` - 62 nature-themed adjectives (e.g., "silent", "ancient", "hidden")
- `NOUNS []string` - 58 nature-themed nouns (e.g., "moon", "river", "breeze")
- Total combinations: 3,596 (62 × 58)

**generator_test.go** (~37 lines)

- `TestNameGenerator` - Tests constructor
- `TestNameGenerator_Generate` - Tests name generation (non-empty string check)
- Uses `time.Now().UTC().UnixNano()` for random seeds in tests

### Data Flow

```
User calls NewNameGenerator(seed)
  → Creates NameGenerator with rand.New(rand.NewSource(seed))
  → Returns as Generator interface

User calls Generate()
  → random.Intn(len(ADJECTIVES)) → select adjective
  → random.Intn(len(NOUNS)) → select noun
  → fmt.Sprintf("%v-%v", adjective, noun)
  → Returns "adjective-noun" string
```

### Performance Characteristics

- **Time Complexity**: O(1) for `Generate()` - two array lookups + string formatting
- **Memory**: ~64 bytes per `NameGenerator` instance (struct + `rand.Rand` state)
- **Word Lists**: Loaded at compile time, shared across all instances (~3.5KB in data segment)
- **No Allocations**: Minimal allocations per call (only the returned string)

## Code Style Conventions

### File Headers

All source files include Apache License 2.0 headers:

```go
// Copyright 2022 [Acorn Labs, Inc.](http://acorn.io)
// Copyright 2018, Goomba project Authors. All rights reserved.
//
// Licensed to the Apache Software Foundation (ASF) under one or more
// contributor license agreements...
```

### Naming

- **Exported**: PascalCase (`Generator`, `NameGenerator`, `NewNameGenerator`)
- **Unexported**: camelCase (`random` field)
- **Package-level constants**: SCREAMING_SNAKE_CASE (`ADJECTIVES`, `NOUNS`)

### Documentation

- Doc comments use abbreviated style: `// TypeName ...` or `// FunctionName ...`
- All exported symbols are documented

### Formatting

- **Indentation**: Tabs (not spaces)
- **Formatting tool**: `go fmt` (standard Go formatting)
- **No external dependencies**: Keep it that way - this is a zero-dependency library

## Important Constraints

### Thread Safety

**CRITICAL**: `NameGenerator` instances are NOT thread-safe. The underlying `rand.Rand` is not safe for concurrent access.

**Safe patterns:**

```go
// ✅ One generator per goroutine
gen1 := namegenerator.NewNameGenerator(seed1)
gen2 := namegenerator.NewNameGenerator(seed2)
go useGenerator(gen1)
go useGenerator(gen2)
```

**Unsafe patterns:**

```go
// ❌ Sharing generator across goroutines
gen := namegenerator.NewNameGenerator(seed)
go gen.Generate()  // RACE CONDITION
go gen.Generate()  // RACE CONDITION
```

### Zero External Dependencies

This library intentionally has **zero external dependencies** to remain lightweight and minimize supply chain risk. Do not add external dependencies without strong justification.

## Testing Strategy

### Current Test Coverage

- Constructor initialization (`TestNameGenerator`)
- Name generation non-empty check (`TestNameGenerator_Generate`)

### Testing Gaps (opportunities for improvement)

- No deterministic seed tests (verify same seed → same sequence)
- No format validation (ensure "adjective-noun" pattern with hyphen)
- No word list validation (ensure ADJECTIVES and NOUNS are non-empty)
- No benchmarks for performance measurement

### How to Add Tests

```go
func TestNameGenerator_DeterministicSequence(t *testing.T) {
    seed := int64(12345)
    gen1 := NewNameGenerator(seed)
    gen2 := NewNameGenerator(seed)

    for i := 0; i < 10; i++ {
        name1 := gen1.Generate()
        name2 := gen2.Generate()
        if name1 != name2 {
            t.Fatalf("Expected deterministic sequence, got %s != %s", name1, name2)
        }
    }
}
```

## Documentation

Comprehensive documentation exists in `docs/`:

- **API.md** - Complete API reference with examples
- **USER_GUIDE.md** - Step-by-step tutorials and common tasks
- **ARCHITECTURE.md** - Design decisions and implementation details
- **CODE_EXAMPLES.md** - 16 practical code examples
- **DIAGRAMS.md** - 18 Mermaid architecture diagrams
- **USAGE_PATTERNS.md** - Advanced integration patterns

The documentation is indexed in **PROJECT_INDEX.md** (human-readable) and **PROJECT_INDEX.json** (machine-readable) for efficient AI context loading (88% token reduction).

## Common Use Cases

1. **Docker container naming** - Temporary, memorable container names
2. **Kubernetes resource naming** - Pod, service, or deployment names
3. **Test data generation** - Unique identifiers in test suites
4. **Temporary file naming** - Human-readable temp file names
5. **Database snapshots** - Memorable snapshot identifiers

## API Surface (Complete)

The library exports exactly **5 symbols**:

1. `Generator` interface - `Generate() string`
2. `NameGenerator` struct - Concrete implementation
3. `NewNameGenerator(seed int64) Generator` - Factory function
4. `ADJECTIVES` - `[]string` of 62 adjectives
5. `NOUNS` - `[]string` of 58 nouns

**Note**: `ADJECTIVES` and `NOUNS` are exported but intended as read-only. Modifying them affects all future generations globally (not thread-safe).

## Module Path Migration Note

The module was originally `github.com/acorn-io/namegenerator` and has been migrated to `github.com/jrmatherly/namegenerator`. License headers and copyright notices reflect both Acorn Labs, Inc. (2022) and Goomba project Authors (2018) origins.

## Workspace Integration

This project is part of the AI workspace. Additional resources:

- **Claude Code commands**: `AI/.claude/commands/` (expert-mode, etc.)
- **Shared agents**: `AI/.claude/agents/` (explore, security-audit, etc.)
- **SuperClaude skills**: `/sc:analyze`, `/sc:test`, `/sc:git`, etc.
- **Serena memories**: `AI/.serena/memories/` (task_completion_checklist, etc.)
- **GitHub Actions**: Workspace-level PR review and issue triage

For session initialization with full context, run `/expert-mode` from the workspace root.
