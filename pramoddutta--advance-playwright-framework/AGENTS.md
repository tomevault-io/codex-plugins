# GitHub Copilot Instructions

## Project Overview
This is a **Playwright Test Automation Framework** using TypeScript with a **3-layer architecture**:
- **Layer 1: Pages** - Locators and basic UI actions only
- **Layer 2: Modules** - Business logic and workflow orchestration
- **Layer 3: Tests** - Test specifications using modules

---

## Architecture Rules

### NEVER Skip Layers
```
Tests → Modules → Pages → Browser
```
Tests should NEVER directly access Page locators. Always go through Modules.

---

## Page Class Rules

When generating Page classes:

1. **Locators as arrow functions** (NOT regular methods):
```typescript
// ✅ CORRECT
submitButton = () => this.page.locator('[data-testid="submit"]');

// ❌ WRONG
getSubmitButton() { return this.page.locator('[data-testid="submit"]'); }
```

2. **Constructor pattern**:
```typescript
constructor(private page: Page) {}
```

3. **Simple actions only** - no business logic, no conditionals:
```typescript
// ✅ CORRECT
async fillUsername(value: string): Promise<void> {
    await this.usernameInput().fill(value);
}

// ❌ WRONG
async fillUsername(value: string): Promise<void> {
    if (value.includes('@')) {  // NO LOGIC!
        await this.emailInput().fill(value);
    }
}
```

4. **File location**: `src/pages/{Feature}Page.ts`

---

## Module Class Rules

When generating Module classes:

1. **Use Page methods** - never use `page.locator()` directly:
```typescript
// ✅ CORRECT
await this.loginPage.fillUsername(username);

// ❌ WRONG
await this.page.locator('#username').fill(username);
```

2. **Include Logger for steps**:
```typescript
this.logger.step(1, 'Navigate to login');
await this.loginPage.navigate();
```

3. **Orchestrate workflows**:
```typescript
async doLogin(username: string, password: string): Promise<boolean> {
    this.logger.step(1, 'Navigate to login page');
    await this.loginPage.navigate();

    this.logger.step(2, 'Enter credentials');
    await this.loginPage.fillUsername(username);
    await this.loginPage.fillPassword(password);

    this.logger.step(3, 'Submit login');
    await this.loginPage.clickLogin();

    return true;
}
```

4. **File location**: `src/modules/{Feature}Module.ts`

---

## Test Spec Rules

When generating test specifications:

1. **Use test.describe() with tags**:
```typescript
test.describe('@P0 @Login Feature Name', () => {
```

2. **Use test.step() for reporting**:
```typescript
await test.step('Login as user', async () => {
    await loginModule.doLogin('user', 'pass');
});
```

3. **Use fixtures for page objects and modules**:
```typescript
test('example', async ({ loginModule, page }) => {
```

4. **File location**: `src/tests/{feature}.spec.ts`

---

## Import Conventions

Always use path aliases:
```typescript
import { LoginPage } from '@pages/LoginPage';
import { LoginModule } from '@modules/LoginModule';
import { Logger } from '@utils/Logger';
import { config } from '@config/index';
```

Available aliases:
- `@pages/*` → `src/pages/*`
- `@modules/*` → `src/modules/*`
- `@utils/*` → `src/utils/*`
- `@fixtures/*` → `src/fixtures/*`
- `@api/*` → `src/api/*`
- `@config/*` → `src/config/*`
- `@testdata/*` → `src/testdata/*`

---

## Common Patterns

### Wait Helpers
```typescript
// Use WaitHelper, not hardcoded timeouts
await waitHelper.waitForCondition(async () => await element.isVisible());

// NEVER do this:
await page.waitForTimeout(5000);
```

### Test Data
```typescript
// Use testdata files
import usersData from '@testdata/users.json';

// NEVER hardcode credentials:
await loginModule.doLogin('admin', 'password123');  // BAD!
```

### Error Messages
```typescript
// Always include meaningful error messages
expect(result).toBeTruthy();  // BAD - no context
expect(result, 'Login should succeed').toBeTruthy();  // GOOD
```

---

## File Naming

| Type | Pattern | Example |
|------|---------|---------|
| Page | `{Feature}Page.ts` | `LoginPage.ts` |
| Module | `{Feature}Module.ts` | `LoginModule.ts` |
| Test | `{feature}.spec.ts` | `login.spec.ts` |
| API | `{Feature}Api.ts` | `AuthApi.ts` |
| Utility | `{Name}Helper.ts` | `WaitHelper.ts` |

---

## Restrictions

- ❌ Never use `console.log` - use Logger
- ❌ Never use hardcoded waits - use explicit waits
- ❌ Never put locators in Modules
- ❌ Never put business logic in Pages
- ❌ Never skip test.step() for important actions
- ❌ Never create duplicate page instances

---
> Source: [PramodDutta/Advance-Playwright-Framework](https://github.com/PramodDutta/Advance-Playwright-Framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
