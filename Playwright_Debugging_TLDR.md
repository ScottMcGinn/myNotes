# Playwright Debugging TL;DR

Quick reference guide for debugging Playwright tests.

## Essential Debug Commands

```powershell
# Run in headed mode (see browser)
npx playwright test --headed

# Run in debug mode (step through)
npx playwright test --debug

# Run with UI mode (interactive debugging)
npx playwright test --ui

# Run specific test file
npx playwright test tests/example.spec.ts

# Run tests matching pattern
npx playwright test -g "login"

# Run in specific browser
npx playwright test --project=chromium
```

## Quick Debugging Techniques

### 1. Pause Execution
```typescript
// Stop and open Playwright Inspector
await page.pause();
```

### 2. Show Browser
```typescript
// In playwright.config.ts
use: {
  headless: false,
  slowMo: 1000, // Slow down by 1 second
}
```

### 3. Take Screenshots
```typescript
await page.screenshot({ path: 'screenshot.png' });
await page.screenshot({ path: 'fullpage.png', fullPage: true });
```

### 4. Console Logging
```typescript
// Log to console
console.log(await page.title());
console.log(await page.locator('.class').textContent());

// Listen to browser console
page.on('console', msg => console.log('BROWSER:', msg.text()));
```

### 5. View Element State
```typescript
// Check if element exists
console.log(await page.locator('.element').count());

// Get element properties
console.log(await page.locator('.element').isVisible());
console.log(await page.locator('.element').isEnabled());
console.log(await page.locator('.element').textContent());
```

## Common Issues & Quick Fixes

### Element Not Found
```typescript
// ❌ BAD - Fails immediately
await page.click('.button');

// ✅ GOOD - Waits up to 30s
await page.locator('.button').click();

// Wait for element to appear
await page.waitForSelector('.element', { state: 'visible' });

// Check if element exists first
const count = await page.locator('.element').count();
if (count > 0) {
  await page.locator('.element').click();
}
```

### Timing Issues
```typescript
// Wait for page to fully load
await page.goto('https://example.com', { waitUntil: 'networkidle' });

// Wait for specific element
await page.waitForSelector('.loaded');

// Wait for URL change
await page.waitForURL('**/dashboard');

// Wait for response
await page.waitForResponse(response => 
  response.url().includes('/api/data') && response.status() === 200
);

// Add explicit wait (use sparingly)
await page.waitForTimeout(1000);
```

### Selector Issues
```typescript
// Use Playwright's auto-waiting locators
await page.locator('text=Submit').click();
await page.locator('button:has-text("Submit")').click();
await page.getByRole('button', { name: 'Submit' }).click();
await page.getByLabel('Username').fill('test');
await page.getByPlaceholder('Enter email').fill('test@test.com');
await page.getByTestId('submit-button').click();
```

### Flaky Tests
```typescript
// Retry failed tests
export default defineConfig({
  retries: 2,
});

// Increase timeout
test.setTimeout(60000); // 60 seconds

// Use soft assertions (don't stop on failure)
await expect.soft(page.locator('.element')).toBeVisible();

// Wait for network to be idle
await page.goto(url, { waitUntil: 'networkidle' });
```

## Debugging Tools

### 1. Playwright Inspector
```powershell
# Interactive debugging
npx playwright test --debug

# Pick locator in browser
npx playwright codegen https://example.com
```

### 2. Trace Viewer
```typescript
// Enable tracing
use: {
  trace: 'on', // or 'on-first-retry', 'retain-on-failure'
}
```

```powershell
# View trace after test
npx playwright show-trace trace.zip
```

### 3. Video Recording
```typescript
use: {
  video: 'on', // or 'retain-on-failure'
}
```

### 4. VS Code Extension
- Install "Playwright Test for VSCode"
- Set breakpoints
- Run/debug tests from editor
- View trace viewer inline

## Quick Inspect Commands

```typescript
// Get page HTML
console.log(await page.content());

// Get element HTML
console.log(await page.locator('.element').innerHTML());

// Get all matching elements
const elements = await page.locator('.items').all();
console.log(`Found ${elements.length} items`);

// Evaluate JavaScript
const result = await page.evaluate(() => {
  return document.querySelector('.element')?.textContent;
});

// Get network requests
page.on('request', request => console.log('>>', request.method(), request.url()));
page.on('response', response => console.log('<<', response.status(), response.url()));
```

## Test Isolation

```typescript
// Use test.beforeEach for setup
test.beforeEach(async ({ page }) => {
  await page.goto('https://example.com');
});

// Use test.afterEach for cleanup
test.afterEach(async ({ page }) => {
  // Clear cookies/storage if needed
  await page.context().clearCookies();
});

// Run tests in isolation
export default defineConfig({
  fullyParallel: true, // Each test gets fresh browser context
});
```

## Useful Config Settings

```typescript
export default defineConfig({
  // Debug mode settings
  use: {
    headless: false,
    slowMo: 500,
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    trace: 'on-first-retry',
  },
  
  // Increase timeouts
  timeout: 60000,
  expect: {
    timeout: 10000,
  },
  
  // Better error messages
  reporter: [
    ['html'],
    ['list'],
  ],
});
```

## Environment-Specific Debugging

```typescript
// Development
if (process.env.DEBUG) {
  use: {
    headless: false,
    slowMo: 1000,
  }
}

// CI
if (process.env.CI) {
  use: {
    trace: 'retain-on-failure',
    video: 'retain-on-failure',
  }
}
```

## Quick Test Patterns

### Debug Single Test
```typescript
test.only('debug this test', async ({ page }) => {
  await page.pause(); // Opens inspector
  // ... test code
});
```

### Skip Flaky Test Temporarily
```typescript
test.skip('flaky test', async ({ page }) => {
  // ... test code
});
```

### Conditional Skip
```typescript
test('desktop only', async ({ page, browserName }) => {
  test.skip(browserName === 'webkit', 'Not working on Safari');
  // ... test code
});
```

### Add Test Annotations
```typescript
test('slow test', async ({ page }) => {
  test.slow(); // Triples the timeout
  // ... test code
});
```

## Network Debugging

```typescript
// Block specific requests
await page.route('**/*.{png,jpg,jpeg}', route => route.abort());

// Mock API response
await page.route('**/api/data', route => {
  route.fulfill({
    status: 200,
    body: JSON.stringify({ success: true }),
  });
});

// Log all network activity
page.on('request', request => {
  console.log('REQUEST:', request.url());
});

page.on('response', response => {
  console.log('RESPONSE:', response.status(), response.url());
});

page.on('requestfailed', request => {
  console.log('FAILED:', request.url());
});
```

## Browser Context Debugging

```typescript
// Get cookies
const cookies = await page.context().cookies();
console.log(cookies);

// Get local storage
const localStorage = await page.evaluate(() => {
  return JSON.stringify(window.localStorage);
});
console.log(localStorage);

// Get session storage
const sessionStorage = await page.evaluate(() => {
  return JSON.stringify(window.sessionStorage);
});
console.log(sessionStorage);
```

## Error Handling

```typescript
// Try/catch for better error messages
try {
  await page.locator('.missing').click({ timeout: 5000 });
} catch (error) {
  console.log('Element not found:', error.message);
  await page.screenshot({ path: 'error.png' });
  throw error;
}

// Soft assertions (continue on failure)
await expect.soft(page.locator('.title')).toHaveText('Expected');
await expect.soft(page.locator('.subtitle')).toBeVisible();
// Test continues even if assertions fail
```

## Quick Checklist for Debugging

1. ✅ Run test in headed mode: `--headed`
2. ✅ Add `await page.pause()` before failing line
3. ✅ Check element selector with Inspector
4. ✅ Verify element is visible: `await page.locator('.el').isVisible()`
5. ✅ Check network tab for failed requests
6. ✅ Increase timeout if needed
7. ✅ Take screenshot before failure
8. ✅ Enable trace viewer for detailed timeline
9. ✅ Check console for JavaScript errors
10. ✅ Verify test runs in isolation (not dependent on other tests)

## One-Liner Debugging Commands

```powershell
# Generate test code
npx playwright codegen https://example.com

# Open last HTML report
npx playwright show-report

# Update Playwright
npm install -D @playwright/test@latest

# Install/update browsers
npx playwright install

# List all installed browsers
npx playwright list

# Show config
npx playwright show-config

# Get system info
npx playwright --version
node --version
npm --version
```

## Pro Tips

- **Use `page.pause()`** - Opens inspector at that exact point
- **Use `test.only()`** - Run just one test quickly
- **Enable UI Mode** - `--ui` for interactive debugging
- **Check `waitUntil`** - Use `networkidle` for dynamic pages
- **Avoid hardcoded waits** - Use `waitForSelector` instead of `waitForTimeout`
- **Use test.step()** - Group actions for better error messages
- **Screenshot on error** - Helps identify what went wrong
- **Use data-testid** - More reliable than CSS selectors
- **Check browser console** - Listen to `page.on('console')`
- **Verify in multiple browsers** - Bug might be browser-specific

## Emergency Debug Template

```typescript
import { test, expect } from '@playwright/test';

test('debug test', async ({ page }) => {
  // Enable console logging
  page.on('console', msg => console.log('BROWSER:', msg.text()));
  
  // Navigate
  await page.goto('https://example.com');
  await page.screenshot({ path: '1-loaded.png' });
  
  // Pause for inspection
  await page.pause();
  
  // Your test steps here
  try {
    await page.locator('.element').click();
  } catch (error) {
    await page.screenshot({ path: 'error.png' });
    console.log('Error:', error.message);
    throw error;
  }
  
  // Final screenshot
  await page.screenshot({ path: 'final.png' });
});
```

## Resources

- [Playwright Docs](https://playwright.dev/docs/intro)
- [API Reference](https://playwright.dev/docs/api/class-playwright)
- [Best Practices](https://playwright.dev/docs/best-practices)
- [Debugging Guide](https://playwright.dev/docs/debug)
- [Trace Viewer](https://playwright.dev/docs/trace-viewer)
