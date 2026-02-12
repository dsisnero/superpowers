---
name: porting-to-crystal
description: Use when porting existing code from another language (especially Go) to Crystal, ensuring idiomatic translation while maintaining exact behavior and logic.
---

# Porting to Crystal

## Overview

This skill provides patterns and guidelines for porting code from other languages (particularly Go) to Crystal while maintaining exact behavior and logic, differing only in Crystal language idioms and standard library usage.

**Core principle:** The Go code is the source of truth. All logic must match the Go implementation, only differing in Crystal language idioms and libraries.

## When to Use

- Setting up a new porting project? Use `initialize-crystal-porting-project` first
- Porting Go packages to Crystal (common in projects like `x/ansi`)
- Translating functions, structs, interfaces, and constants
- Converting Go tests to Crystal specs
- Deciding between exact translation vs idiomatic Crystal patterns
- Ensuring type safety and performance in Crystal

**When NOT to use:**
- Writing new Crystal code from scratch
- Porting between languages with fundamentally different paradigms (e.g., Python to Crystal)
- When you can't verify behavior against the original implementation

**Decision Flow:**
1. **Porting Go to Crystal?** → Use this skill
2. **Go code is source of truth?** → Must maintain exact logic
3. **Can verify behavior?** → Must have tests (port Go tests)
4. **Changing logic for "idiomatic" reasons?** → Stop, logic must match exactly

## Core Patterns

### Module Structure

Go packages become Crystal modules. Place constants, classes, and methods inside a module matching the Go package name.

```crystal
# Go: package ansi
module Ansi
  # ...
end
```

### Constants

Go `const` blocks become Crystal constants with explicit types. Use `_u8`, `_u32`, etc. suffixes for numeric literals.

```crystal
# Go: const NUL = 0x00
NUL = 0x00_u8
```

Group related constants in nested modules (e.g., `C0`, `C1`).

### Functions and Methods

Go functions become Crystal class methods (`def self.method_name`). Use snake_case for method names.

```crystal
# Go: func SetForegroundColor(s string) string
def self.set_foreground_color(s : String) : String
  # ...
end
```

Preserve parameter order and semantics. Use Crystal splat (`*args`) for Go variadic parameters.

### Types and Interfaces

Go interfaces become Crystal modules or abstract structs. Go structs become Crystal structs with getters.

```crystal
# Go: type BasicColor uint8
struct BasicColor
  getter value : UInt8
  # ...
end
```

### Error Handling

Go's multiple return values (value, error) become Crystal's exception handling or `Nil` return. Use `nil` for simple error cases.

### Escape Sequences

Use `\e` for escape (`\x1b` in Go). Use `\a` for bell (`\x07`).

### Arrays and Slices

Go slices are references to underlying arrays, similar to Crystal's `Slice` type. Use `Slice(T)` for performance-critical code, C interop, or when you need pointer semantics. For byte slices (`[]byte`), use `Bytes` (alias for `Slice(UInt8)`). Use `Array` for convenience when you need dynamic resizing or higher-level operations. Go arrays with fixed size become Crystal `StaticArray`.

- `[]byte` → `Bytes` (preferred) or `Slice(UInt8)`
- `[]T` → `Slice(T)` (performance) or `Array(T)` (convenience)
- `[N]T` → `StaticArray(T, N)`

### Maps

Go maps become Crystal `Hash`. Specify key and value types.

### Testing

Go test files (`*_test.go`) become Crystal spec files (`*_spec.cr`). Use `describe` and `it` blocks. Preserve exact test logic and assertions.

```crystal
# Go: func TestSetHyperlink(t *testing.T)
describe "SetHyperlink" do
  it "creates hyperlink with no params" do
    # assertion
  end
end
```

## Quick Reference

| Go Construct | Crystal Equivalent |
|--------------|-------------------|
| `package ansi` | `module Ansi` |
| `const NUL = 0x00` | `NUL = 0x00_u8` |
| `func F() string` | `def self.f : String` |
| `type S struct { R uint8 }` | `struct S; getter r : UInt8; end` |
| `type I interface { M() }` | `module I; abstract def m; end` |
| `[]byte` | `Bytes` (alias for `Slice(UInt8)`) - preferred for binary data |
| `[]T` | `Slice(T)` (performance) or `Array(T)` (convenience) |
| `map[string]int` | `Hash(String, Int32)` |
| `panic("error")` | `raise "error"` |
| `t.Errorf("msg")` | `fail "msg"` |
| `\x1b` | `\e` |
| `\x07` | `\a` |
| `strings.Join(params, ":")` | `params.join(":")` |

## Implementation

### Step 1: Understand the Go Code

Read the Go source thoroughly. Identify dependencies, interfaces, and external packages. Check if Crystal equivalents exist (e.g., `colorful` for `go-colorful`).

### Step 2: Create Crystal Module Structure

Create a `.cr` file with the same base name as the Go file. Define the module and require dependencies.

### Step 3: Port Constants and Types

Translate constants with appropriate Crystal types. Define structs with getters and initializers.

### Step 4: Port Functions

Convert functions to class methods with snake_case names. Preserve parameter semantics. Add type annotations.

### Step 5: Port Tests

Create a `*_spec.cr` file. Convert each test function to `it` blocks. Keep assertions identical to Go.

### Step 6: Verify Behavior

Run Crystal specs and compare output with Go test results. Ensure edge cases match.

### Step 7: Apply Crystal Idioms

After ensuring logic matches, apply Crystal idioms: use `?` for nullable, `!` for raising, `Enumerable` methods, etc.

## Common Mistakes

1. **Forgetting module wrapper** - Functions must be inside `module Ansi` (or appropriate module).
2. **Wrong integer types** - Use `UInt8`, `Int32`, etc. with suffix literals.
3. **Missing type annotations** - Crystal needs explicit types for method parameters and return values.
4. **Incorrect escape sequences** - Use `\e` not `\x1b`.
5. **Skipping test porting** - Every Go test must have a Crystal spec counterpart.
6. **Changing logic for "idiomatic" reasons** - Logic must match Go exactly; only syntax changes.
7. **Ignoring error handling** - Go's error returns must be represented (nil, exception, or union type).
8. **Using Crystal's `String` for binary data** - Use `Bytes` (or `Slice(UInt8)`) for `[]byte`.
9. **Using `Array` for performance-critical slices** - Consider `Slice` for better performance when working with C interop or large data.

## Red Flags

These thoughts mean STOP—you're rationalizing:

| Thought | Reality |
|---------|---------|
| "This is simple, I can just write Crystal directly" | Even simple functions have subtle differences in escape sequences, integer types, and module structure |
| "I don't need to check the Go code, I understand the functionality" | The Go code is the source of truth; assumptions can introduce behavioral differences |
| "I'll adapt this to be more idiomatic Crystal" | Logic must match Go exactly; only syntax should change to Crystal idioms |
| "I can skip porting tests for now" | Every Go test must have a Crystal spec counterpart to ensure behavioral equivalence |
| "The Go implementation seems wrong, I'll fix it in Crystal" | Don't fix perceived bugs; port exactly. File issue upstream if needed |
| "I'll use Array instead of Slice for convenience" | Performance and semantics matter; use Slice/Bytes for []byte, Array for dynamic collections |
| "I'll just copy-paste and adjust syntax" | Copy-paste leads to missed type annotations and incorrect integer literals |
| "This function doesn't need a module wrapper" | All ported code must be inside the appropriate module for namespace consistency |
| "I can use String for binary data, it's fine" | []byte must become Bytes (Slice(UInt8)) to preserve binary semantics |
| "I'll port later, let me work on something else first" | Unported code creates gaps; follow systematic porting steps |
| "Crystal has a better library for this, I'll use that instead of porting the Go dependency" | Dependencies must be ported or matched exactly; using different libraries changes behavior |
| "This Go package is too complex, I'll skip it" | All Go code must be ported; complexity is not a reason to omit functionality |

## Rationalization Table

Add explicit counters to common rationalizations:

| Rationalization | Counter |
|----------------|---------|
| "This is trivial" | Trivial code often has subtle differences (escape sequences, integer types, module structure) |
| "I know Crystal better than Go" | The Go code is the source of truth; assumptions lead to behavioral divergence |
| "I'll make it more Crystal-like" | Logic must match exactly; only apply Crystal idioms after verifying equivalence |
| "Tests can wait" | Tests ensure correctness; port them first or alongside implementation |
| "The Go code seems buggy" | Port exactly; report bugs upstream; don't "fix" in port |
| "Array is easier" | Use Slice/Bytes for []byte; Array for dynamic collections requiring resizing |
| "I'll just copy-paste" | Copy-paste misses type annotations, integer suffixes, and module wrappers |
| "Module wrapper is unnecessary" | Crystal requires module namespace for organization and avoiding conflicts |
| "String works for binary data" | Bytes (Slice(UInt8)) preserves binary semantics; String may encode/escape |
| "I'll do this later" | Systematic porting prevents gaps; follow the implementation steps |
| "Crystal has a better library for this, I'll use that instead of porting the Go dependency" | Dependencies must be ported or matched exactly; using different libraries changes behavior |
| "This Go package is too complex, I'll skip it" | All Go code must be ported; complexity is not a reason to omit functionality |

## See Also

- [initialize-crystal-porting-project](../initialize-crystal-porting-project) - Project setup for porting
- [systematic-debugging](../systematic-debugging) - Debugging test failures
- [verification-before-completion](../verification-before-completion) - Quality gates before completion
- [test-driven-development](../test-driven-development) - TDD for feature implementation

## Real-World Impact

Following these patterns ensures:

- **Behavioral equivalence** - Crystal code behaves exactly like Go code
- **Maintainability** - Crystal idioms make code readable and maintainable
- **Test coverage** - All original tests are preserved
- **Performance** - Crystal's type system and compilation produce efficient code
- **Cross-language consistency** - Changes in Go can be ported systematically