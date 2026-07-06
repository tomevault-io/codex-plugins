# AI Coding Agent Instructions for Projectsdp

## Project Overview
This is a structured Java learning project with daily problem-solving exercises (day1, day2, etc.). Each day contains multiple programming challenges with increasing complexity, from basic I/O to object-oriented design patterns.

## Project Structure & Architecture

### Core Organization
```
src/
├── App.java                    # Entry point (boilerplate)
├── day1/                       # Basic programming challenges
│   ├── Ascii.java             # I/O operations
│   ├── Cricket.java           # Rectangle calculations
│   ├── News.java
│   └── conditional_stmt/      # Conditional logic puzzles
│       ├── KepricanNum.java   # Number theory (Kaprekar numbers)
│       ├── MinTravelTime.java
│       └── WatwrBill.java
└── day2/                       # OOP and advanced algorithms
    ├── StudentData.java       # Array of objects, aggregations
    ├── Bank.java              # OOP with getters/setters
    ├── Encrypt.java           # Algorithm (non-prime digit sums)
    ├── Empdata.java
    └── Quiz.java

bin/                           # Compiled output (auto-generated)
lib/                           # External dependencies (jar files)
```

### No Dependencies
This project has no external libraries. All code uses only `java.util` and `java.io` standard library.

## Development Workflow

### Compilation & Execution
- **Build**: VSCode Java extension auto-compiles on save to `bin/` folder
- **Run**: Use VSCode's "Run" button on any class with `main()` method, or press `Ctrl+F5` on the file
- **Java Version**: Configured in VSCode settings (see `.vscode/settings.json`)

### Build Configuration
- Source path: `src/`
- Output path: `bin/`
- Referenced libraries: `lib/**/*.jar` (currently empty)

## Code Patterns & Conventions

### 1. Problem-Based Structure
Each file represents a **single programming challenge** with:
- Problem statement in Javadoc or block comments at the top
- Example input/output in comments
- Self-contained `main()` method with Scanner for user input

**Example** (`day1/conditional_stmt/KepricanNum.java`):
```java
/*
Kaprekar number explanation
Input/Output examples...
*/
public class KepricanNum {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        // solution code
    }
}
```

### 2. Package Naming
- `day1`, `day2`, `day1.conditional_stmt` - organize by learning progression
- Each package represents a learning milestone
- Nested packages (e.g., `day1.conditional_stmt`) group related problem types

### 3. Input/Output Pattern
All programs use **Scanner for input** and **System.out.println for output**:
```java
Scanner sc = new Scanner(System.in);
int input = sc.nextInt();
System.out.println(result);
sc.close(); // cleanup when done
```

### 4. OOP Introduction (day2)
- `Bank.java` shows getter/setter pattern with inner class `BankAccount`
- `StudentData.java` demonstrates array-of-objects for managing collections
- Objects encapsulate related data (e.g., account holder + balance)

### 5. Algorithm Patterns
- **String manipulation**: `Long.toString()` and `substring()` for digit analysis (KepricanNum, Encrypt)
- **Conditional logic**: Prime/non-prime classification (Encrypt.java)
- **Aggregation**: Finding max values and summing properties (StudentData.java)

## When Adding New Files

1. **Follow day-based organization**: Place in `src/day{N}/` folder
2. **Include problem statement**: Add comments at the top with problem description and examples
3. **Use Scanner/System.out**: Match existing I/O conventions
4. **Create bin/ subdirectories**: Compiled classes auto-generate in `bin/day{N}/` structure
5. **Package declaration**: Always match the source folder structure (e.g., `package day2;`)

## Critical Developer Notes
- **No main method found**: If a class has no `main()`, it's meant as a helper class (see `BankAccount` in `Bank.java`)
- **Resource cleanup**: Call `scanner.close()` when done (see `Encrypt.java`)
- **Long vs int**: Use `long` for large numbers (number theory problems like Kaprekar)
- **String-based arithmetic**: Problems often require converting numbers to strings for digit-level manipulation

---
> Source: [SmithaAarya/Skill_Development_java](https://github.com/SmithaAarya/Skill_Development_java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-06 -->
