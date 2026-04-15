# Project general coding standards

## Naming Conventions
- Use camelCase for variables, functions, and methods for C++ code
- Use snake_case for variables, functions, and methods for C code
- In C and C++ always append opening { on a new line
- In c++ lambda functions, always use { on a new line
- Use ALL_CAPS for constants in #define in C
- Use PascalCase for class names in C++
- All comments should be in English and should be clear and concise and only used when necessary
- In if, for, while, and switch statements, single-line statements should not use braces, but multi-line statements should always use braces
- In C++ constructor implementations, put : at the end of the function declaration on the first line. Add all member initialization on a new line with the , at the end of each line.
Example:
MyClass::MyClass():
    MyBaseClass(a, b),
    member_1(3),
    member_2(nullptr)
{
}

## Error Handling
- Always log errors with contextual information
- Do not use exceptions in C++ code, use return codes instead

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/calaos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
