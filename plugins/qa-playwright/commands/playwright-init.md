---
description: Initialize a new Playwright project with best practices
---

# Playwright Project Initialization

Set up a new Playwright E2E testing project following best practices and standards.

## Input
Ask the user to provide provide:
- Target application URL (optional, defaults to http://localhost:3000)
- Existing project context (framework, auth requirements, etc.)
- Specific browser requirements

## Process

### Step 1: Assess Current State

Check for existing Playwright setup:
1. Look for `playwright.config.ts` or `playwright.config.js`
2. Check `package.json` for `@playwright/test` dependency
3. Look for existing `tests/`, `e2e/`, or `apps/` folders

**If Playwright already exists:**
Ask the user: "Playwright is already configured. Would you like me to:
1. Review and upgrade to recommended standards (preserves existing tests)
2. Start fresh (will backup existing setup)"

**If Playwright does not exist:**
Ask the user for more context and information such as the target application URL and any other needed information to get started.

### Step 2: Choose Project Structure

Ask the user: "What type of test suite structure do you need?"

**Option 1: Single-app** - Testing one application
```
{project}/
├── playwright.config.ts
├── playwright-commands.md
├── CLAUDE.md
├── package.json
├── shared/
│   ├── fixtures/index.ts
│   ├── pages/base.page.ts
│   └── utils/test-helpers.ts
└── tests/
    └── e2e/
        └── smoke.spec.ts
```

**Option 2: Multi-app** - Testing multiple applications or domains
```
{project}/
├── playwright.config.ts
├── playwright-commands.md
├── CLAUDE.md
├── package.json
├── shared/
│   ├── config/
│   ├── fixtures/index.ts
│   ├── pages/base.page.ts
│   └── utils/test-helpers.ts
└── apps/
    └── {app-name}/
        ├── pages/
        ├── test-data/
        └── tests/
            └── smoke.spec.ts
```

For multi-app, also ask: "What is the name of your first application?" (e.g., "gateway", "client-portal", "web-app")

### Step 3: Install Playwright

Launch the `playwright-engineer` agent to execute:

```bash
# Initialize Playwright (if not present)
npm init playwright@latest -- --yes --quiet

# Install accessibility testing
npm install -D @axe-core/playwright
```

### Step 4: Create Folder Structure

Based on the user's choice, create the appropriate structure.

**For single-app:**
- `shared/fixtures/`
- `shared/pages/`
- `shared/utils/`
- `tests/e2e/`

**For multi-app:**
- `shared/config/`
- `shared/fixtures/`
- `shared/pages/`
- `shared/utils/`
- `apps/{app-name}/pages/`
- `apps/{app-name}/test-data/`
- `apps/{app-name}/tests/`

### Step 5: Configure playwright.config.ts

**Single-app configuration:**
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['junit', { outputFile: 'test-results/results.xml' }],
    ['list']
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
    actionTimeout: 15000,
    navigationTimeout: 30000,
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],
});
```

**Multi-app configuration:**
```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './apps',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: [
    ['html', { open: 'never' }],
    ['junit', { outputFile: 'test-results/results.xml' }],
    ['list']
  ],
  use: {
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
    actionTimeout: 15000,
    navigationTimeout: 30000,
  },
  projects: [
    // {app-name} projects
    {
      name: '{app-name}-chromium',
      use: {
        ...devices['Desktop Chrome'],
        baseURL: process.env.{APP_NAME}_URL || 'http://localhost:3000',
      },
      testDir: './apps/{app-name}/tests',
    },
    {
      name: '{app-name}-firefox',
      use: {
        ...devices['Desktop Firefox'],
        baseURL: process.env.{APP_NAME}_URL || 'http://localhost:3000',
      },
      testDir: './apps/{app-name}/tests',
    },
    {
      name: '{app-name}-webkit',
      use: {
        ...devices['Desktop Safari'],
        baseURL: process.env.{APP_NAME}_URL || 'http://localhost:3000',
      },
      testDir: './apps/{app-name}/tests',
    },
  ],
});
```

### Step 6: Create Base Page Object

```typescript
// shared/pages/base.page.ts
import { Page, Locator } from '@playwright/test';

export abstract class BasePage {
  constructor(protected readonly page: Page) {}

  abstract get url(): string;

  async goto(): Promise<void> {
    await this.page.goto(this.url);
  }

  async waitForPageLoad(): Promise<void> {
    await this.page.waitForLoadState('networkidle');
  }

  async waitForDomContentLoaded(): Promise<void> {
    await this.page.waitForLoadState('domcontentloaded');
  }

  getByTestId(testId: string): Locator {
    return this.page.getByTestId(testId);
  }

  getByRole(role: Parameters<Page['getByRole']>[0], options?: Parameters<Page['getByRole']>[1]): Locator {
    return this.page.getByRole(role, options);
  }

  getByLabel(text: string | RegExp, options?: { exact?: boolean }): Locator {
    return this.page.getByLabel(text, options);
  }

  getByPlaceholder(text: string | RegExp, options?: { exact?: boolean }): Locator {
    return this.page.getByPlaceholder(text, options);
  }

  getByText(text: string | RegExp, options?: { exact?: boolean }): Locator {
    return this.page.getByText(text, options);
  }

  async getPageTitle(): Promise<string> {
    return this.page.title();
  }

  async getCurrentUrl(): Promise<string> {
    return this.page.url();
  }

  async takeScreenshot(name: string): Promise<Buffer> {
    return this.page.screenshot({ path: `test-results/${name}.png` });
  }
}
```

### Step 7: Create Fixtures and Helpers

**Fixtures:**
```typescript
// shared/fixtures/index.ts
import { test as base } from '@playwright/test';
import AxeBuilder from '@axe-core/playwright';

export { expect } from '@playwright/test';

export const test = base.extend<{ makeAxeBuilder: () => AxeBuilder }>({
  makeAxeBuilder: async ({ page }, use) => {
    await use(() => new AxeBuilder({ page }));
  },
});
```

**Test Helpers:**
```typescript
// shared/utils/test-helpers.ts
import { Page } from '@playwright/test';

export async function waitForNetworkIdle(page: Page, timeout = 5000): Promise<void> {
  await page.waitForLoadState('networkidle', { timeout });
}

export async function mockApiResponse(
  page: Page,
  urlPattern: string | RegExp,
  response: unknown,
  status = 200
): Promise<void> {
  await page.route(urlPattern, async route => {
    await route.fulfill({
      status,
      contentType: 'application/json',
      body: JSON.stringify(response),
    });
  });
}

export function generateTestId(prefix: string): string {
  return `${prefix}-${Date.now()}-${Math.random().toString(36).substring(7)}`;
}

export async function waitForApiRequest(
  page: Page,
  urlPattern: string | RegExp,
  timeout = 10000
): Promise<void> {
  await page.waitForRequest(urlPattern, { timeout });
}

export async function clearBrowserStorage(page: Page): Promise<void> {
  await page.evaluate(() => {
    localStorage.clear();
    sessionStorage.clear();
  });
}

export async function captureConsoleErrors(page: Page): Promise<string[]> {
  const errors: string[] = [];
  page.on('console', msg => {
    if (msg.type() === 'error') {
      errors.push(msg.text());
    }
  });
  return errors;
}
```

### Step 8: Create Smoke Test

**Single-app location:** `tests/e2e/smoke.spec.ts`
**Multi-app location:** `apps/{app-name}/tests/smoke.spec.ts`

```typescript
import { test, expect } from '@playwright/test';

test.describe('Smoke Tests', () => {
  test('should load the application', async ({ page }) => {
    await page.goto('/');
    await expect(page).toHaveTitle(/.+/);
  });

  test('should have no console errors on page load', async ({ page }) => {
    const errors: string[] = [];
    page.on('console', msg => {
      if (msg.type() === 'error') {
        errors.push(msg.text());
      }
    });

    await page.goto('/');
    await page.waitForLoadState('networkidle');

    expect(errors).toEqual([]);
  });
});
```

### Step 9: Generate Documentation

Create two documentation files in the project root.

#### playwright-commands.md

Create a command reference file with these sections:

1. **Quick Reference** - Table of common npm commands
2. **Project Structure** - Diagram matching the chosen structure (single-app or multi-app)
3. **Running Tests** - Commands for all tests, by browser, by app (if multi-app)
4. **UI Mode & Debugging** - UI mode and debug mode commands with feature lists
5. **Direct CLI Options** - Running specific tests, workers, trace/screenshots
6. **Configuration Details** - Timeouts table, CI/CD behavior table
7. **Common Workflows** - Daily testing, debugging, generating new tests

#### CLAUDE.md

Create a project context file with these sections:

1. **Overview** - Brief description of the test suite
2. **Project Structure** - Diagram matching the chosen structure
3. **Key Patterns**
   - Page Objects: base class location, inheritance pattern, locator methods
   - Fixtures: import path, available fixtures (makeAxeBuilder)
   - Test Organization: describe blocks, data-driven testing, accessibility tests
4. **File Naming Conventions** - Table with patterns for .spec.ts, .page.ts, .data.ts
5. **Related Documentation** - Link to playwright-commands.md with maintenance instructions:

   > **Keep playwright-commands.md updated when:**
   > - Adding new npm scripts to package.json
   > - Adding new test projects or applications
   > - Changing environment configuration
   > - Modifying browser project settings in playwright.config.ts

6. **npm Scripts** - JSON block with all test scripts to add to package.json

### Step 10: Update .gitignore

Add Playwright artifacts:

```
# Playwright
/test-results/
/playwright-report/
/blob-report/
/playwright/.cache/
```

### Step 11: Verify Setup

1. Run the smoke tests: `npx playwright test`
2. Confirm tests pass
3. Show the HTML report: `npx playwright show-report`

## Important Guidelines

- **Check existing setup first** - Don't overwrite custom configuration without asking
- **Preserve existing tests** - Migrate, don't delete
- **Verify before completing** - Tests must run successfully
- **Ask about base URL** - Confirm the application URL with the user
- **Keep documentation in sync** - CLAUDE.md references playwright-commands.md for maintenance
