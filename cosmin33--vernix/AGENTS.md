# Vernix - Scala Mini Language Interpreter

Vernix is an interpreter for a Scala-like mini programming language built with Scala 3.7.2, using ZIO, cats, fastparse, and spire libraries. The project provides a DSL for creating programs that can be compiled to expressions and executed.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Setup
- Install SBT (Scala Build Tool) manually since package repositories may have network issues:
  ```bash
  wget https://github.com/sbt/sbt/releases/download/v1.10.6/sbt-1.10.6.zip
  unzip sbt-1.10.6.zip
  export PATH=$PATH:$(pwd)/sbt/bin
  ```
- Verify installation: `sbt --version`

### Build and Compile
- **CRITICAL**: Compilation takes ~30 seconds. NEVER CANCEL. Use timeout of 60+ seconds.
  ```bash
  sbt compile
  ```
- Package the project (~5 seconds):
  ```bash
  sbt package
  ```
- Expected output: `target/scala-3.7.2/vernix_3-0.1.0-SNAPSHOT.jar`

### Running Applications
The project has two main classes:

1. **Expr01** (Primary demonstration application):
   ```bash
   sbt "runMain io.vernix.Expr01"
   ```
   - Demonstrates the Vernix DSL with variables, functions, and compilation
   - Expected output: Shows program stages and result: `28`
   - Runtime: ~6 seconds

2. **Parsing** (Parser development, currently incomplete):
   ```bash
   sbt "runMain io.vernix.Parsing"
   ```
   - Expected output: `Failure(scala.NotImplementedError: an implementation is missing)`
   - Runtime: ~5 seconds

### Interactive Development
- **Scala Console**: Test Vernix DSL interactively
  ```bash
  sbt console
  ```
  Example usage:
  ```scala
  import io.vernix.*
  val test = Program.value(42)
  // Exit with :quit
  ```

### Testing
- **No tests currently exist** in the project
- Run test command (no-op): `sbt test`

## Validation Scenarios

### End-to-End Validation
After making changes, ALWAYS validate the core functionality:

1. **Compile the project**: `sbt compile` - must succeed
2. **Run the main demo**: `sbt "runMain io.vernix.Expr01"` - verify output shows result `28`
3. **Test console access**: Start `sbt console`, import packages, create a simple Program
4. **Verify package generation**: `sbt package` - JAR should be created

### Manual Testing Workflow
```bash
# Full validation sequence (takes ~45 seconds total)
sbt compile && sbt "runMain io.vernix.Expr01" && sbt package
```

## Build System Details

### Key Commands and Timing
- `sbt compile` - 30 seconds - NEVER CANCEL
- `sbt package` - 5 seconds
- `sbt "runMain <class>"` - 5-6 seconds
- `sbt console` - 10 seconds to start
- `sbt test` - 1-5 seconds (no tests exist)

### SBT Configuration
- Scala version: 3.7.2
- SBT version: 1.11.5 (via 1.10.6 binaries)
- Main dependencies: ZIO 2.1.17, ZIO-HTTP 3.0.1, cats-core 2.9.0, fastparse 3.1.1
- Test framework: ZIO Test (configured but no tests exist)

### Available SBT Tasks
Key tasks you can use:
- `compile` - Compile source code
- `package` - Create JAR file
- `run` / `runMain` - Execute main classes
- `console` - Start Scala REPL with project classpath
- `clean` - Clean build artifacts
- `doc` - Generate API documentation

## Codebase Architecture

### Core Components
Located in `src/main/scala/io/vernix/`:

- **programs.scala**: Core `Program[A]` trait and DSL operations
- **expressions.scala**: `Expr[A]` trait for compiled expressions
- **statements.scala**: `Statements[F[_]]` trait extending `Ops[F]`
- **ops.scala**: `Ops[F[_]]` trait defining core operations with ZIO and Try implementations
- **Type.scala**: Type system with given instances for basic types
- **OpContext.scala**: Context management for variables and functions
- **Parsing.scala**: Parser implementation (incomplete - returns NotImplementedError)
- **Expr01.scala**: Main ZIO application demonstrating the interpreter

### Key Entry Points
- **Main demos**: `io.vernix.Expr01` (working), `io.vernix.Parsing` (incomplete)
- **Console imports**: Start with `import io.vernix.*` to access all DSL components

### Development Patterns
- Use `Program.value(x)` to create constant programs
- Use `Program.variable[T]("name")` for variable references
- Chain operations with `*>` (sequence) and arithmetic operators
- Compile programs with `.compile` returning `Try[Expr[A]]`
- Execute expressions with `.apply[F]` where F is Task, Try, etc.

## Troubleshooting

### Common Issues
- **SBT not found**: Install manually using wget method above
- **Compilation warnings**: Safe to ignore deprecation warnings about `_` vs `?` wildcards
- **JAR won't run directly**: Use `sbt runMain` instead of `java -jar` (missing classpath)
- **Network issues**: SBT repositories may be inaccessible, use manual installation

### Build Artifact Locations
- Compiled classes: `target/scala-3.7.2/classes/`
- JAR file: `target/scala-3.7.2/vernix_3-0.1.0-SNAPSHOT.jar`
- Build cache: `target/scala-3.7.2/zinc/`

### No Linting/Formatting
- **No scalafmt or other formatters configured**
- **No linting tools available** 
- Code follows standard Scala 3 formatting conventions

## Common Tasks Reference

### Repository Root Contents
```
├── build.sbt              # SBT build configuration
├── project/
│   └── build.properties   # SBT version: 1.11.5
├── src/main/scala/io/vernix/  # Main source code
├── target/                 # Build artifacts (gitignored)
├── README.md              # "Interpreter for a scala-like mini programming language"
└── .gitignore             # Standard Scala/IntelliJ exclusions
```

### Example Workflow
```bash
# 1. Setup (first time only)
wget https://github.com/sbt/sbt/releases/download/v1.10.6/sbt-1.10.6.zip
unzip sbt-1.10.6.zip
export PATH=$PATH:$(pwd)/sbt/bin

# 2. Development cycle
sbt compile                    # 30 seconds - NEVER CANCEL
sbt "runMain io.vernix.Expr01" # Verify functionality

# 3. Interactive testing
sbt console
import io.vernix.*
val prog = Program.value(42) + Program.value(8)
:quit

# 4. Package for distribution
sbt package
```

### Performance Expectations
- **Initial SBT startup**: 10-15 seconds (downloads dependencies)
- **Incremental compilation**: 5-10 seconds after first compile
- **Application execution**: 5-6 seconds including SBT overhead
- **Total development cycle**: ~45 seconds for full build+test+package

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cosmin33)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cosmin33)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
