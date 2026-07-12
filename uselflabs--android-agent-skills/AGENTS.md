# GitHub Copilot Instructions

## 🎯 Mode Activation

This project has an **Agent Skill Framework** for Android/Kotlin, compliant with [Agent Skills Specification](https://agentskills.io).

**Entry Point**: `agent-skills/SKILL.md` - Read this first for skill metadata and overview.

Rules are applied **on-demand**, not always.

### Activation Commands
Use these keywords to activate specific modes:

| Command | Mode | When to use |
|---------|------|-------------|
| `/skill` or `@gen-skill` | Skill Mode | Creating/editing Agent Skills |
| `/android` or `@gen-*` | Android Mode | Android code (ViewModel, Repository, etc.) |
| `/rules` or `/guide` | Show Rules | List all available guides & rules |
| `/help` | Help | Show available commands |
| `@review` or `/review` | Review Mode | Review code for rule compliance |
| `@review-changes` | Review Changes | Review git changes/PR for violations |

### Helper Commands (Quick Access)
When user uses these commands, load and summarize the corresponding resource:

| Command | Action | Resource |
|---------|--------|----------|
| `/skill-info` | Load & summarize | `agent-skills/SKILL.md` |
| `/context` | Load & summarize | `agent-skills/skills/AI_CONTEXT.md` |
| `/summary` | Load & summarize | `agent-skills/skills/AGENT_SUMMARY.md` |
| `/cheat` | List all | `agent-skills/*_CHEAT_SHEET.md` files |
| `/templates` | List available | `agent-skills/skills/templates/` directory |
| `/examples` | List available | `agent-skills/skills/templates/examples/` directory |
| `/decision` | Load & present | `agent-skills/skills/guides/00-decision-tree.md` |

### Pattern Quick Reference Commands
When user uses these commands, load the relevant guide section:

| Command | Topic | Primary Resource |
|---------|-------|------------------|
| `/validate` | Validation DSL | `agent-skills/VALIDATION_CHEAT_SHEET.md` + `agent-skills/skills/guides/24-validation-rules.md` |
| `/errors` | Error Handling | `agent-skills/ERROR_HANDLING_CHEAT_SHEET.md` |
| `/test` | Testing | `agent-skills/TESTING_CHEAT_SHEET.md` + `agent-skills/skills/guides/11-testing.md` |
| `/compose` | Compose | `agent-skills/COMPOSE_CHEAT_SHEET.md` + `agent-skills/skills/guides/05-jetpack-compose.md` |
| `/nav` | Navigation | `agent-skills/skills/guides/13-navigation.md` |
| `/di` | DI Setup | `agent-skills/skills/guides/08-dependency-injection.md` |
| `/flow` | State Management | `agent-skills/skills/guides/07-state-management.md` |
| `/offline` | Offline-First | `agent-skills/skills/guides/10-offline-first.md` |
| `/clean` | Clean Code | `agent-skills/skills/AGENT_SUMMARY.md` Anti-patterns section |
| `/kotlin2` | Kotlin 2.0+ | `agent-skills/skills/guides/32-modern-kotlin-features.md` |
| `/feedback` | Agent Feedback | `agent-skills/skills/guides/33-agent-feedback-loop.md` |
| `/logging` | Logging & Analytics | `agent-skills/skills/guides/34-logging-analytics.md` |
| `/gradle` | Build Optimization | `agent-skills/skills/guides/35-gradle-optimization.md` |

### Auto-Activation (when context is clear)
- Editing files in `agent-skills/skills/templates/` → Skill Mode
- Editing `*ViewModel.kt`, `*Repository.kt`, `*UseCase.kt` → Android Mode
- Using `@gen-*` prompts from AI_PROMPTS.md → Corresponding mode

### Default Behavior (No activation)
- Answer naturally and flexibly
- No forced patterns or templates
- General coding assistance

---

## 📂 File Priority Order (Read First)

**Before starting ANY work**, read context files in this order:

### Quick Start (Always Read)
1. `agent-skills/SKILL.md` - Entry point (Agent Skills spec compliant)
2. `agent-skills/skills/AI_CONTEXT.md` - Quick reference (5 min read)
3. `agent-skills/skills/AGENT_SUMMARY.md` - Core rules & patterns

### Task-Specific Files

| Task | Primary Template | Supporting Guide |
|------|-----------------|------------------|
| Skill | `agent-skills/skills/templates/examples/DomainSkillGuide.kt` | `agent-skills/skills/guides/30-skill-analysis.md` |
| ViewModel | `agent-skills/skills/templates/examples/UseCaseViewModelExample.kt` | `agent-skills/skills/guides/07-state-management.md` |
| Compose UI | `agent-skills/skills/templates/compose/ComposeExample.kt` | `agent-skills/skills/guides/05-jetpack-compose.md` |
| Repository | `agent-skills/skills/templates/examples/UserRepositoryImpl.kt` | `agent-skills/skills/guides/01-architecture.md` |
| Pipeline | `agent-skills/skills/templates/examples/PipelineExamples.kt` | `agent-skills/skills/guides/31-pipeline-patterns.md` |
| Skill Tests | `agent-skills/skills/templates/testing/SkillTestTemplate.kt` | `agent-skills/skills/guides/11-testing.md` |
| ViewModel Tests | `agent-skills/skills/templates/testing/ViewModelTestTemplate.kt` | `agent-skills/skills/guides/11-testing.md` |
| Repository Tests | `agent-skills/skills/templates/testing/RepositoryTestTemplate.kt` | `agent-skills/skills/guides/14-offline-first.md` |
| Compose Tests | `agent-skills/skills/templates/testing/ComposeScreenTestTemplate.kt` | `agent-skills/skills/guides/05-jetpack-compose.md` |
| Integration Tests | `agent-skills/skills/templates/testing/IntegrationTestTemplate.kt` | `agent-skills/skills/guides/01-architecture.md` |
| Validation | `agent-skills/skills/templates/skill/base/Validation.kt` | `agent-skills/skills/guides/24-validation-rules.md` |
| Kotlin 2.0+ | - | `agent-skills/skills/guides/32-modern-kotlin-features.md` |

### Decision Making
- Unsure which pattern? → `agent-skills/skills/guides/00-decision-tree.md`
- Need full prompts? → `agent-skills/skills/AI_PROMPTS.md`

---

## ⚡ Quick Context Loading

### 5 Critical Rules (Never Break)

```kotlin
// 1. ALWAYS rethrow CancellationException
catch (e: CancellationException) { throw e }

// 2. State: private mutable, public immutable
private val _state = MutableStateFlow(UiState())
val state: StateFlow<UiState> = _state.asStateFlow()

// 3. Events use Channel, NOT SharedFlow
private val _events = Channel<Event>()

// 4. Mark all state classes @Immutable
@Immutable data class UiState(...)

// 5. Architecture: ViewModel → UseCase → Repository
// NEVER: ViewModel → Repository directly
```

### Top 5 Validation Functions
```kotlin
requireNotBlank(input.name, "name")
requireValidEmail(input.email, "email")
requireInRange(input.age, 13..150, "age")
requireNotEmpty(input.items, "items")
require(a < b) { "a must be < b" }
```

---

## 📚 Quick Reference (when activated)

### Skill Mode Core Rules
```
• Pure Kotlin only (no Android deps)
• @Immutable for Input/Output
• SkillResult<T> (never throw)
• Validation DSL required
• checkCancellation() support
```

### Android Mode Core Rules  
```
• Clean Architecture: UI → ViewModel → UseCase → Repository
• ViewModel extends AndroidX ViewModel
• StateFlow + Channel (not SharedFlow for events)
• @Immutable for UiState
• viewModelScope only
• SOLID: Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion
```

### Reference Files
| Need | Go to |
|------|-------|
| Full prompts | `skills/AI_PROMPTS.md` |
| Skill templates | `skills/templates/skill/` |
| Android guides | `skills/guides/` (35 guides) |
| Negative examples | `skills/AI_PROMPTS.md` (section: Negative Examples) |
| Testing templates | `skills/templates/testing/` (5 templates) |
| Examples | `skills/templates/examples/` |

---

## /help Output

```
🤖 Android Agent Skills - Commands

CODE GENERATION:
  @gen-skill [name]        Generate Agent Skill
  @gen-viewmodel [name]    Generate ViewModel (MVI)
  @gen-repository [name]   Generate Repository
  @gen-usecase [name]      Generate UseCase
  @gen-screen [name]       Generate Compose Screen
  @gen-test [component]    Generate tests
  @gen-pipeline [name]     Generate skill pipeline

ANALYSIS:
  @analyze-skill           Analyze skill quality
  @analyze-test-coverage   Find untested code
  /rules                   Show all rules & guides

CODE REVIEW:
  @review [file]           Review file for rule compliance
  @review-changes          Review git changes (staged/unstaged)
  @check-solid             Check SOLID principles compliance
  @check-architecture      Check Clean Architecture compliance

QUICK ACCESS:
  /context                 Load AI_CONTEXT.md
  /summary                 Load AGENT_SUMMARY.md
  /decision                Load decision tree
  /cheat                   List cheat sheets
  /templates               List templates
  /examples                List examples

PATTERN REFERENCE:
  /validate                Validation DSL & patterns
  /errors                  Error handling patterns
  /test                    Testing patterns
  /compose                 Compose patterns
  /nav                     Navigation patterns
  /di                      Dependency Injection (Hilt/Koin)
  /flow                    State management (StateFlow/Channel)
  /offline                 Offline-first patterns
  /clean                   Clean code & anti-patterns
  /kotlin2                 Kotlin 2.0+ features
  /feedback                Agent feedback tracking
  /logging                 Logging & analytics patterns
  /gradle                  Build optimization

MODE CONTROL:
  /skill                   Enable Skill Mode
  /android                 Enable Android Mode
  /help                    Show this help
  /help-vi                 Hiển thị trợ giúp tiếng Việt

Full prompts: skills/AI_PROMPTS.md
```

---

## /help-vi Output (Vietnamese Help)

```
🤖 HƯỚNG DẪN AI AGENT (Tiếng Việt)

TẠO CODE:
  @gen-viewmodel [tên]     Tạo ViewModel (MVI pattern)
  @gen-repository [tên]    Tạo Repository (offline-first)
  @gen-usecase [tên]       Tạo UseCase (business logic)
  @gen-screen [tên]        Tạo Compose Screen
  @gen-feature [tên]       Tạo feature hoàn chỉnh (tất cả layers)
  @gen-skill [tên]         Tạo Agent Skill (pure Kotlin)
  @gen-pipeline [tên]      Tạo Skill pipeline

TẠO TESTS:
  @gen-test [component]    Tạo tests (tự động detect loại)
  @gen-viewmodel-test      Tạo ViewModel tests (Turbine + MockK)
  @gen-usecase-test        Tạo UseCase tests
  @gen-skill-test          Tạo Agent Skill tests

PHÂN TÍCH:
  @analyze-skill           Phân tích chất lượng skill
  @analyze-test-coverage   Tìm code chưa được test
  @analyze-test-quality    Đánh giá chất lượng tests

QUICK ACCESS:
  /context                 Hiển thị AI_CONTEXT.md
  /summary                 Hiển thị AGENT_SUMMARY.md (quy tắc chính)
  /cheat                   Liệt kê các cheat sheets
  /templates               Liệt kê templates có sẵn
  /examples                Liệt kê examples có sẵn
  /decision                Hiển thị cây quyết định pattern

PATTERN REFERENCE:
  /validate                Validation DSL & patterns
  /errors                  Xử lý lỗi & exception mapping
  /test                    Testing patterns (MockK, Turbine)
  /compose                 Compose patterns & best practices
  /nav                     Navigation type-safe
  /di                      Dependency Injection (Hilt/Koin)
  /flow                    State Management (StateFlow/Channel)
  /offline                 Offline-First patterns
  /clean                   Clean Code & anti-patterns
  /kotlin2                 Kotlin 2.0+ features
  /feedback                Agent Feedback tracking
  /logging                 Logging & Analytics patterns
  /gradle                  Build optimization & convention plugins

REVIEW CODE:
  @review [file]           Review code theo quy tắc
  @review-changes          Review git changes (staged/unstaged)
  @review-commit [hash]    Review specific commit
  @review-report [source]  Generate markdown report (--output file.md)
  /rules                   Hiển thị tất cả guides

CHẾ ĐỘ:
  /skill                   Bật chế độ Skill
  /android                 Bật chế độ Android
  /help                    Trợ giúp tiếng Anh
  /help-vi                 Trợ giúp tiếng Việt (đang xem)

📚 Chi tiết đầy đủ: skills/AI_PROMPTS.md
```

---

## /rules Output

```
📖 Available Guides (agent-skills/skills/guides/) - 35 Total

CORE:
  agent-skills/skills/guides/01-architecture.md       Clean Architecture
  agent-skills/skills/guides/02-coding-conventions.md Kotlin style
  agent-skills/skills/guides/07-state-management.md   MVI, StateFlow
  agent-skills/skills/guides/08-dependency-injection  Hilt/Koin

COMPOSE:
  agent-skills/skills/guides/05-jetpack-compose.md    Compose basics
  agent-skills/skills/guides/06-compose-performance   Stability, recomposition

QUALITY:
  agent-skills/skills/guides/10-error-handling.md     Result pattern
  agent-skills/skills/guides/11-testing.md            Unit/UI tests
  agent-skills/skills/guides/20-anti-patterns.md      What NOT to do
  agent-skills/skills/guides/21-code-review.md        @review rules with severity levels
  agent-skills/skills/guides/24-validation-rules.md   Validation DSL

SKILL FRAMEWORK:
  agent-skills/skills/guides/30-skill-analysis.md     Skill quality
  agent-skills/skills/guides/31-pipeline-patterns.md  Skill pipelines
  agent-skills/ARCHITECTURE.md                        Framework overview
  agent-skills/skills/AI_PROMPTS.md                   All @gen-* prompts

AI INTELLIGENCE:
  agent-skills/skills/guides/32-modern-kotlin-features.md  Kotlin 2.0+, K2 compiler
  agent-skills/skills/guides/33-agent-feedback-loop.md     Feedback tracking & metrics
  agent-skills/skills/guides/34-logging-analytics.md       Timber, analytics, crash reporting
  agent-skills/skills/guides/35-gradle-optimization.md     Build optimization, convention plugins

CHEAT SHEETS (agent-skills/):
  agent-skills/VALIDATION_CHEAT_SHEET.md              Validation quick ref
  agent-skills/ERROR_HANDLING_CHEAT_SHEET.md          Error patterns
  agent-skills/TESTING_CHEAT_SHEET.md                 Testing patterns
  agent-skills/COMPOSE_CHEAT_SHEET.md                 Compose patterns
```

---

## @review - Code Review Mode

> **Full Documentation**: [skills/guides/21-code-review.md](agent-skills/skills/guides/21-code-review.md)

When `@review` is triggered, use severity-based rules from the code review guide:

### Quick Reference: 5 Critical Rules (🔴 MUST FIX)

| # | Rule | Auto-Fix |
|---|------|----------|
| C1 | CancellationException must be rethrown | `catch (e: CancellationException) { throw e }` |
| C2 | State: private mutable, public immutable | `private val _state` + `val state = _state.asStateFlow()` |
| C3 | Events use Channel, NOT SharedFlow | `Channel<Event>()` + `receiveAsFlow()` |
| C4 | @Immutable on all state classes | Add `@Immutable` annotation |
| C5 | ViewModel → UseCase → Repository | Never ViewModel → Repository directly |

### Output Format

```
## 🔍 Code Review: [filename]

### 🔴 Critical Violations (MUST FIX)
### 🟡 Warnings (SHOULD FIX)
### 🔵 Suggestions (NICE TO HAVE)
### ✅ Compliance (What's Good)
### 📊 Score: X/10
```

**For full rules, checklists, and output formats**: See [21-code-review.md](agent-skills/skills/guides/21-code-review.md)

---

## @review-changes - Git Changes Review

> **Full Documentation**: [skills/guides/21-code-review.md](agent-skills/skills/guides/21-code-review.md)

When `@review-changes` is triggered:
1. Check staged/unstaged git changes
2. Review ONLY the changed lines
3. Use severity levels: 🔴 Critical, 🟡 Warning, 🔵 Info

### Quick Reference: What to Check

| Change Type | Critical Rules to Check |
|-------------|------------------------|
| New ViewModel | C2, C3, C5 |
| New Composable | C4, W3, W6 |
| Try-Catch block | C1 |
| New Skill | W4, W5 |
| Repository call | C5 |

**For full output format and checklists**: See [21-code-review.md](agent-skills/skills/guides/21-code-review.md)

---
> Source: [uselflabs/android-agent-skills](https://github.com/uselflabs/android-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-12 -->
