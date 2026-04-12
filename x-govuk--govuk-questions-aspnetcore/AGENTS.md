# Copilot Instructions for govuk-questions-aspnetcore

## Building and Testing

### Build
To build the solution, use:
```bash
just --configuration Release
```

### Test
To test the solution, run:
```bash
just test --configuration Release
```

### Code Formatting
Any code changes should be formatted with:
```bash
just format
```

## Testing Guidelines

### Test Structure
All tests should have clear 'Arrange', 'Act' and 'Assert' sections to improve readability and maintainability.

### Exception Testing
For tests that throw exceptions, instead of combining the 'Act' and 'Assert' into a single call (e.g., `Assert.Throws`), prefer capturing the exception in the 'Act' section and asserting in the 'Assert' section.

**Preferred pattern:**
```csharp
// Arrange
var sut = new MyClass();

// Act
var exception = Record.Exception(() => sut.MethodThatThrows());

// Assert
Assert.NotNull(exception);
Assert.IsType<InvalidOperationException>(exception);
Assert.Equal("Expected error message", exception.Message);
```

**For async methods:**
```csharp
// Arrange
var sut = new MyClass();

// Act
var exception = await Record.ExceptionAsync(async () => await sut.MethodThatThrowsAsync());

// Assert
Assert.NotNull(exception);
Assert.IsType<InvalidOperationException>(exception);
Assert.Equal("Expected error message", exception.Message);
```

**Avoid combining Act and Assert:**
```csharp
// Don't do this:
var exception = Assert.Throws<InvalidOperationException>(() => sut.MethodThatThrows());
Assert.Equal("Expected error message", exception.Message);
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/x-govuk)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/x-govuk)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
