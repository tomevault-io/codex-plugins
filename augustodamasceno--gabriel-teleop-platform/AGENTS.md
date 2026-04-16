# Role: Senior Computer Engineer  
# Goal: Generate high-performance, idempotent, and strictly formatted code.

## Contact
> email: augustodamasceno@protonmail.com

## 1. Global Standards (Apply to All Languages)

### Copyright Headers
**Rule:** Insert the following header at the very top of every new file.
- **Year:** Current Year (e.g., 2026).
- **Author:** Augusto Damasceno.
- **License:** Use `BSD-2-Clause` for Open Source, `LicenseRef-<BUSINESS>-Proprietary` for Closed Source.

**Template (Open Source):**
```text
/*
 * FILE DESCRIPTION
 *
 * Copyright (c) 2026, Augusto Damasceno.
 * All rights reserved.
 *
 * SPDX-License-Identifier: BSD-2-Clause
 */
```

**Template (Closed Source):**
```text
/*
 * FILE DESCRIPTION
 *
 * Copyright (c) 2026, <BUSINESS>.
 * All rights reserved.
 *
 * SPDX-License-Identifier: LicenseRef-<BUSINESS>-Proprietary
 * This software is proprietary and confidential.
 */
```

### Indentation & Formatting
- **Indentation:** STRICTLY 4 spaces. No tabs.
- **Idempotency:** Ensure functions are pure where possible. Retries must not duplicate side effects. Handle errors gracefully.

---

## 2. Language Rules: C / C++

### Code Style
- **Brace Style:** Allman Style (Braces always on a new line).
- **Switch Statements:** Align `case` labels vertically with the `switch` keyword.
- **Include Guards:** Use `#ifndef _PROJECT_FILE_H` format.

### Naming Conventions
| Entity | Case / Style | Example |
| :--- | :--- | :--- |
| **Files** | snake_case (lowercase) | `ecc_curve_defs.h` |
| **Private Members** | camelCase + trailing `_` | `value_` |
| **Macros** | UPPER_SNAKE_CASE | `CACHE_SIZE_DEFAULT` |
| **Variables** | snake_case | `item_width` |
| **Functions** | snake_case | `string_size()` |
| **Methods** | snake_case | `reset_geometry()` |
| **Constants** | PascalCase | `Fps` |
| **Structs** | PascalCase | `MovingCorrelation` |
| **Enums** | UPPER_SNAKE_CASE (Elements) | `BMP`, `JPEG` |
| **Classes** | PascalCase | `URLSecurityManager` |
| **Namespaces** | lowercase (start w/ project) | `projectplot` |

### Snippet: Switch Alignment
```c
switch (suffix)
{
case 'G': // Aligned with switch
    mem <<= 30;
    break;
default:
    break;
}
```

---

## 3. Language Rules: C#

### Code Style
- **Standard:** Follow Microsoft C# Coding Conventions.
- **Brace Style:** Allman Style.
- **NinjaTrader Scripts:** property `[NinjaScriptProperty]` must be placed at the very end of the class.

### Class Layout Sequence
1. Fields (Private)
2. Properties (Public)
3. Constructors
4. Methods
5. NinjaScript Properties (if applicable)

### Naming Conventions
| Entity | Case / Style | Example |
| :--- | :--- | :--- |
| **Class/Struct** | PascalCase | `OrderCalculator` |
| **Method** | PascalCase | `CalculateOrder()` |
| **Property** | PascalCase | `TotalAmount` |
| **Interface** | PascalCase | `IOrderService` |
| **Enum** | PascalCase | `OrderStatus` |
| **Variable** | camelCase | `orderTotal` |
| **Parameter** | camelCase | `quantity` |
| **Constant** | PascalCase | `MaxRetryCount` |

---

## 4. Language Rules: Python

- **Standard:** STRICTLY follow PEP 8.
- **Naming:** snake_case for variables/functions, PascalCase for classes.

---

## 5. Output Format: Markdown

- **Structure:** Use Headings (`#`) for hierarchy.
- **Lists:** Use Bullet points (`-`) for rules.
- **Tables:** Use for style conventions or data comparison.
- **Code:** Always wrap code in triple backticks with language tags (e.g., `cpp`).
- **Quotes:** Do NOT use blockquotes (`>`) unless quoting a specific text source.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/augustodamasceno) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-13 -->
