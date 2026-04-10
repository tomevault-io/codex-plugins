# Copilot Instructions for json-zig

This document provides comprehensive information about the json-zig project for GitHub Copilot and other AI assistants.

## 🎯 Project Overview

A complete JSON parser implementation ported from TypeScript to Zig 0.15.1, demonstrating faithful architectural patterns while leveraging Zig's memory safety and performance characteristics.

## ✨ Features & Capabilities

- **🔍 Complete JSON Parsing** - Supports all JSON types: strings, numbers, booleans, null, arrays, objects
- **🛡️ Memory Safe** - Proper allocation/deallocation with no memory leaks
- **⚡ High Performance** - Zig's zero-cost abstractions and compile-time optimizations
- **🎯 Standards Compliant** - Follows JSON specification from [json.org](https://www.json.org/json-en.html)
- **🧪 Thoroughly Tested** - Comprehensive unit test suite with 68 test cases
- **📝 Unicode Support** - Full UTF-8 and Unicode escape sequence handling
- **🔧 CLI Tool** - Command-line interface for parsing JSON from stdin
- **🤖 CI/CD Ready** - GitHub Actions workflow with multi-optimization testing

## 🏗️ Architecture Details

### Modular Design

The implementation follows the same modular architecture as the original TypeScript version:

1. **Tokenization**: Each JSON type has its own parser module
2. **Recursive Descent**: Value parser dispatches to appropriate type parsers
3. **Memory Management**: Explicit allocation/deallocation with proper cleanup
4. **Error Propagation**: Zig's error union system for robust error handling

### Project Structure

```
json-zig/
├── .github/           # GitHub configuration
│   └── workflows/     # CI/CD workflows
│       └── ci.yml     # GitHub Actions CI pipeline
├── lib/src/           # Core JSON parsing library
│   ├── main.zig       # Main parser, types, and tests
│   ├── string.zig     # String parsing (escapes, Unicode)
│   ├── number.zig     # Number parsing (int, float, scientific)
│   ├── value.zig      # Value dispatcher (core router)
│   ├── pair.zig       # Key-value pair parsing
│   ├── array.zig      # Array parsing
│   └── object.zig     # Object parsing
├── cli/src/           # Command-line interface
│   └── main.zig       # CLI implementation
├── typescript/        # Original TypeScript implementation
├── build.zig          # Build configuration
└── README.MD          # User documentation
```

## 🧪 Test Coverage Details

The project includes comprehensive tests covering:

- ✅ **Number parsing**: Integers, floats, negatives, scientific notation, zero, large numbers, small decimals
- ✅ **String parsing**: Basic strings, empty strings, escape sequences, Unicode escapes, control characters
- ✅ **Boolean values**: `true` and `false`
- ✅ **Null values**: `null`
- ✅ **Arrays**: Empty arrays, single elements, multiple elements, extra whitespace handling
- ✅ **Objects**: Empty objects, single pairs, multiple pairs, nested structures, whitespace handling
- ✅ **Whitespace handling**: Leading/trailing whitespace throughout all types
- ✅ **Error cases**: Invalid characters, incomplete JSON, malformed syntax, invalid escapes

**Test Count**: 68 comprehensive unit tests  
**Test Performance**: ~1-2ms execution time  
**Memory Usage**: <2MB RSS during testing

## 🎯 Supported JSON Features

| Feature    | Support | Implementation Notes                                     |
| ---------- | ------- | -------------------------------------------------------- |
| Numbers    | ✅      | Integers, floats, scientific notation                    |
| Strings    | ✅      | UTF-8, escape sequences (\n, \t, \", \\, \/, \b, \f, \r) |
| Unicode    | ✅      | \uXXXX escape sequences with proper UTF-8 encoding       |
| Booleans   | ✅      | `true`, `false` literals                                 |
| Null       | ✅      | `null` literal                                           |
| Arrays     | ✅      | Nested arrays, mixed types, comma separation             |
| Objects    | ✅      | Nested objects, string keys, key-value pairs             |
| Whitespace | ✅      | Flexible whitespace handling (spaces, tabs, newlines)    |

## 🐛 Error Handling System

The parser provides detailed error information through Zig's error union system:

```zig
pub const JsonError = error{
    InvalidInput,           // Empty or null input
    UnexpectedCharacter,    // Invalid character in JSON
    IncompleteExpression,   // Truncated JSON
    OutOfMemory,           // Memory allocation failed
};
```

## 🔧 Development Guidelines

### Code Style Standards

- Follow Zig standard naming conventions (`snake_case` for functions/variables, `PascalCase` for types)
- Use explicit error handling with `try` and error unions
- Include comprehensive tests for new functionality
- Ensure proper memory management with allocators
- Prefer stack allocation where possible, heap allocation when necessary
- Use `defer` for guaranteed cleanup

### Memory Management Patterns

```zig
// Standard allocation pattern
var result = try parse(allocator, json_string);
defer result.deinit(allocator);  // Always cleanup

// ArrayList pattern (Zig 0.15.1)
var list = ArrayList(ValueToken).init(allocator);
defer list.deinit();

// Error handling with cleanup
var result = parse(allocator, input) catch |err| {
    // Cleanup already handled by individual parsers
    return err;
};
```

### Adding New Features

1. **Implement** in the appropriate module under `lib/src/`
2. **Test** comprehensively with unit tests
3. **Document** in code comments and update README if needed
4. **Validate** with `zig build test` and memory leak checks
5. **Performance** test with realistic JSON samples

### Debugging Tips

- Use `std.debug.print()` for debugging output
- Run tests with `--summary all` to see detailed timing
- Use `zig build test --verbose` for build debugging
- Memory issues: Check `deinit()` calls and allocator usage

## 🚀 Performance Characteristics

- **Parse Speed**: ~1-2ms for typical JSON documents
- **Memory Usage**: <2MB RSS for standard operations
- **Allocation Strategy**: Minimal heap allocations, prefer stack when possible
- **Zero-Copy**: String references where safe, copies only when necessary

## 📖 Technical References

- **JSON Specification**: [json.org](https://www.json.org/json-en.html)
- **Zig 0.15.1 Documentation**: [ziglang.org/documentation](https://ziglang.org/documentation/)
- **Zig ArrayList API**: Uses unmanaged collections with explicit allocator passing
- **Unicode Standard**: UTF-8 encoding with \uXXXX escape sequence support

## 🔍 Common Patterns in Codebase

### Parser Module Structure

```zig
// Standard parser module pattern
pub fn parse(allocator: Allocator, expression: []const u8, pos: *usize) JsonError!?ValueToken {
    // Implementation with proper error handling
}

pub fn deinit(self: *SomeType, allocator: Allocator) void {
    // Cleanup implementation
}
```

### Testing Patterns

```zig
test "descriptive test name" {
    var result = try parse(testing.allocator, "test_input");
    defer result.deinit(testing.allocator);

    // Assertions using std.testing
    try testing.expect(condition);
}
```

### Error Handling Patterns

```zig
// Propagate errors up the call stack
return JsonError.UnexpectedCharacter;

// Handle specific errors
const result = parse(...) catch |err| switch (err) {
    JsonError.InvalidInput => return null,
    else => return err,
};
```

This documentation provides comprehensive context for AI assistants working with the json-zig codebase, covering architecture, patterns, testing, and development practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joechung2008)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/joechung2008)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
