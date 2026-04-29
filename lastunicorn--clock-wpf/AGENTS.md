## General

- When you have a better implementation idea (from the technical point of view) than the one that user requested in the prompt, provide your suggestion and wait for the user to choose how to proceeed.

## Code Conventions

- Do not use `var` keyword. Whenever possible, use the actual type.
- When using Linq, prefer using the name `x` for the parameter representing the item.
- When instantiating objects, prefer using the `new()` approach.
- When using the object initializer syntax, if there are more than one property to be initialized, write each property initialization on a different line.
- Do not use the curly brackets for `if`, `for` and `using` statements when the body is a single line of code.

## Code Documentation

- Do not create xml documentation for the types that are used only inside the current solution.
- Only create xml documentation for public types that are exposed as a NuGet package.

## Unit Tests

- When using `Assert.Throws` method, always use a block body for the lambda expression.

- For each public method that is tested (including the constructor), create a different test file.
  - Ex: For a method called `Query()` create test file `QueryTests`
- All the test files for a single class should be placed in a directory containing the name of the class.
  - Ex: For a class named `Color` create directory `ColorTests`.
- Use the naming pattern `Having<...>_When<...>_Then<...>` for the tests. Where:
  - `Having` describes the most important setup details.
  - `When` describes the action tested.
  - `Then` describes the expected result.

## WPF

- Avoid writing code in the code behind class of a window or user control. Instead, create attached behaviors.

---
> Source: [lastunicorn/clock-wpf](https://github.com/lastunicorn/clock-wpf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
