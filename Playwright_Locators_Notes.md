# Playwright Locators - Study Notes

Comprehensive notes from the official Playwright Locators documentation.

## What are Locators?

Locators are the central piece of Playwright's **auto-waiting** and **retry-ability**. They represent a way to find element(s) on the page at any moment.

**Key Principle:** Every time a locator is used for an action, an up-to-date DOM element is located in the page. If the DOM changes due to re-render, the new element corresponding to the locator will be used.

```typescript
const locator = page.getByRole('button', { name: 'Sign in' });
await locator.hover();  // Locates element
await locator.click();  // Locates element again (fresh lookup)
```

## Recommended Built-in Locators (Priority Order)

### 1. **page.getByRole()** - HIGHEST PRIORITY ⭐
Locate by explicit and implicit accessibility attributes.

```typescript
await page.getByRole('button', { name: 'Sign in' }).click();
await page.getByRole('checkbox', { name: 'Subscribe' }).check();
await page.getByRole('heading', { name: 'Sign up' }).toBeVisible();
```

**When to use:** This is the closest way to how users and assistive technology perceive the page. Prioritize this method.

**Common roles:**
- `button`
- `checkbox`
- `heading`
- `link`
- `listitem`
- `textbox`
- `table`
- Many more from [W3C ARIA roles](https://www.w3.org/TR/wai-aria-1.2/#roles)

**Important:** HTML elements like `<button>` have implicitly defined roles that are automatically recognized.

### 2. **page.getByLabel()** - For Form Fields
Locate form controls by their associated label's text.

```typescript
await page.getByLabel('Password').fill('secret');
await page.getByLabel('User Name').fill('John');
```

**HTML Example:**
```html
<label>Password <input type="password" /></label>
```

**When to use:** For form fields with labels.

### 3. **page.getByPlaceholder()** - For Inputs
Locate inputs by placeholder attribute.

```typescript
await page.getByPlaceholder('name@example.com').fill('playwright@microsoft.com');
```

**HTML Example:**
```html
<input type="email" placeholder="name@example.com" />
```

**When to use:** For form elements without labels but with placeholder text.

### 4. **page.getByText()** - For Non-Interactive Elements
Find elements by text content (substring, exact, or regex).

```typescript
// Substring match
await expect(page.getByText('Welcome, John')).toBeVisible();

// Exact match
await expect(page.getByText('Welcome, John', { exact: true })).toBeVisible();

// Regular expression
await expect(page.getByText(/welcome, [A-Za-z]+$/i)).toBeVisible();
```

**Important:** Text matching always normalizes whitespace (multiple spaces → one space, line breaks → spaces, trims whitespace).

**When to use:** For non-interactive elements like `div`, `span`, `p`. For interactive elements (`button`, `a`, `input`), use role locators.

### 5. **page.getByAltText()** - For Images
Locate images by their alt text.

```typescript
await page.getByAltText('playwright logo').click();
```

**HTML Example:**
```html
<img alt="playwright logo" src="/img/playwright-logo.svg" width="100" />
```

**When to use:** For `img` and `area` elements with alt text.

### 6. **page.getByTitle()** - By Title Attribute
Locate elements with a matching title attribute.

```typescript
await expect(page.getByTitle('Issues count')).toHaveText('25 issues');
```

**HTML Example:**
```html
<span title='Issues count'>25 issues</span>
```

**When to use:** When elements have the `title` attribute.

### 7. **page.getByTestId()** - Most Resilient
Locate elements by `data-testid` attribute (or custom attribute).

```typescript
await page.getByTestId('directions').click();
```

**HTML Example:**
```html
<button data-testid="directions">Itinéraire</button>
```

**When to use:** 
- Most resilient method - survives text/role changes
- Use when role/text is not important
- When you can't locate by role or text

**Custom test ID attribute:**
```typescript
// playwright.config.ts
export default defineConfig({
  use: {
    testIdAttribute: 'data-pw'  // Use data-pw instead of data-testid
  }
});
```

### 8. **page.locator()** - CSS/XPath (AVOID IF POSSIBLE)
Use CSS or XPath selectors.

```typescript
await page.locator('css=button').click();
await page.locator('xpath=//button').click();
await page.locator('button').click();  // Auto-detects CSS
await page.locator('//button').click();  // Auto-detects XPath
```

**⚠️ WARNING:** CSS and XPath can break when DOM structure changes. Avoid long chains like:
```typescript
// ❌ BAD - Fragile locator
await page.locator('#tsf > div:nth-child(2) > div.A8SBwf > div.RNNXgb > div > div.a4bIc > input').click();
```

**When to use:** Only as a last resort. Prefer user-facing locators.

## Shadow DOM Support

✅ All locators work with Shadow DOM by default
❌ Exceptions:
- XPath does not pierce shadow roots
- Closed-mode shadow roots are not supported

```typescript
// Works even with shadow DOM
await page.getByText('Details').click();

// Ensure element contains text
await expect(page.locator('x-details')).toContainText('Details');
```

## Filtering Locators

### Filter by Text
```typescript
// Filter list items by text
await page
  .getByRole('listitem')
  .filter({ hasText: 'Product 2' })
  .getByRole('button', { name: 'Add to cart' })
  .click();

// Filter by regex
await page
  .getByRole('listitem')
  .filter({ hasText: /Product 2/ })
  .getByRole('button', { name: 'Add to cart' })
  .click();
```

### Filter by NOT Having Text
```typescript
// Find items that are in stock
await expect(page.getByRole('listitem').filter({ hasNotText: 'Out of stock' })).toHaveCount(5);
```

### Filter by Child/Descendant
```typescript
// Find product 2's add to cart button
await page
  .getByRole('listitem')
  .filter({ has: page.getByRole('heading', { name: 'Product 2' }) })
  .getByRole('button', { name: 'Add to cart' })
  .click();

// Assert only one product card
await expect(
  page.getByRole('listitem').filter({ has: page.getByRole('heading', { name: 'Product 2' }) })
).toHaveCount(1);
```

**Important:** The filtering locator is relative to the original locator, not the document root.

```typescript
// ❌ WRONG - Won't work
await expect(
  page
    .getByRole('listitem')
    .filter({ has: page.getByRole('list').getByText('Product 2') })
).toHaveCount(1);
```

### Filter by NOT Having Child/Descendant
```typescript
await expect(
  page.getByRole('listitem').filter({ hasNot: page.getByText('Product 2') })
).toHaveCount(1);
```

## Locator Operators

### Matching Inside a Locator (Chaining)
```typescript
const product = page.getByRole('listitem').filter({ hasText: 'Product 2' });
await product.getByRole('button', { name: 'Add to cart' }).click();
await expect(product).toHaveCount(1);

// Chain locators together
const saveButton = page.getByRole('button', { name: 'Save' });
const dialog = page.getByTestId('settings-dialog');
await dialog.locator(saveButton).click();
```

### Matching Two Locators Simultaneously (.and())
```typescript
const button = page.getByRole('button').and(page.getByTitle('Subscribe'));
```

### Matching ONE of Two Alternatives (.or())
```typescript
const newEmail = page.getByRole('button', { name: 'New' });
const dialog = page.getByText('Confirm security settings');

await expect(newEmail.or(dialog).first()).toBeVisible();

if (await dialog.isVisible()) {
  await page.getByRole('button', { name: 'Dismiss' }).click();
}
await newEmail.click();
```

**⚠️ Note:** If both elements are visible, `.or()` will match both and may throw strictness error. Use `.first()` to match only one.

### Matching Only Visible Elements
```typescript
// Click only the visible button
await page.locator('button').filter({ visible: true }).click();
```

**⚠️ Note:** It's usually better to find a more reliable way to identify the element instead of checking visibility.

## Working with Lists

### Count Items in a List
```typescript
await expect(page.getByRole('listitem')).toHaveCount(3);
```

### Assert All Text in a List
```typescript
await expect(page.getByRole('listitem')).toHaveText(['apple', 'banana', 'orange']);
```

### Get Specific Item

#### By Text
```typescript
await page.getByText('orange').click();
```

#### By Filter
```typescript
await page.getByRole('listitem').filter({ hasText: 'orange' }).click();
```

#### By Test ID
```typescript
await page.getByTestId('orange').click();
```

#### By Position (nth, first, last)
```typescript
const banana = await page.getByRole('listitem').nth(1);  // Second item (0-indexed)
const first = await page.getByRole('listitem').first();
const last = await page.getByRole('listitem').last();
```

**⚠️ Warning:** Use with caution. Page changes can make the locator point to a different element. Prefer unique locators.

### Chaining Filters
```typescript
const rowLocator = page.getByRole('listitem');

await rowLocator
  .filter({ hasText: 'Mary' })
  .filter({ has: page.getByRole('button', { name: 'Say goodbye' }) })
  .screenshot({ path: 'screenshot.png' });
```

### Rare Use Cases with Lists

#### Iterate Elements
```typescript
// Using for...of
for (const row of await page.getByRole('listitem').all()) {
  console.log(await row.textContent());
}

// Using regular for loop
const rows = page.getByRole('listitem');
const count = await rows.count();
for (let i = 0; i < count; ++i) {
  console.log(await rows.nth(i).textContent());
}
```

#### Evaluate in Page
```typescript
const rows = page.getByRole('listitem');
const texts = await rows.evaluateAll(list => list.map(element => element.textContent));
```

## Strictness

Playwright locators are **strict** by default.

### Throws Error if More Than One Match
```typescript
// ❌ Throws if multiple buttons exist
await page.getByRole('button').click();
```

### Works Fine with Multiple Elements
```typescript
// ✅ Works with multiple matches
await page.getByRole('button').count();
```

### Opt-out of Strictness (Not Recommended)
```typescript
await page.getByRole('button').first().click();
await page.getByRole('button').last().click();
await page.getByRole('button').nth(0).click();
```

**⚠️ Not recommended:** When your page changes, you may click on an unintended element. Create unique locators instead.

## Chaining and Combining

### Available on Locator and FrameLocator
All `getBy*` methods are available on:
- `page`
- `Locator`
- `FrameLocator`

```typescript
const locator = page
  .frameLocator('#my-frame')
  .getByRole('button', { name: 'Sign in' });
await locator.click();
```

## Code Generation

Use the [code generator](https://playwright.dev/docs/codegen) to generate locators, then edit as needed:

```powershell
npx playwright codegen https://example.com
```

## Best Practices Summary

1. **Prioritize role locators** - Closest to user perception
2. **Use test IDs for resilience** - When role/text might change
3. **Avoid CSS/XPath chains** - Fragile and breaks easily
4. **Filter instead of nth()** - More reliable selection
5. **Chain locators** - Narrow down step by step
6. **Use exact matches carefully** - Whitespace is always normalized
7. **Prefer visible over hidden checks** - Find better unique identifiers
8. **Respect strictness** - Create unique locators instead of using first/last

## Quick Reference

```typescript
// Role (BEST)
page.getByRole('button', { name: 'Submit' })

// Label (Forms)
page.getByLabel('Email')

// Placeholder (Inputs)
page.getByPlaceholder('Enter email')

// Text (Non-interactive)
page.getByText('Welcome')

// Alt text (Images)
page.getByAltText('Logo')

// Title attribute
page.getByTitle('Tooltip text')

// Test ID (Most resilient)
page.getByTestId('submit-btn')

// CSS/XPath (LAST RESORT)
page.locator('button')
page.locator('//button')

// Filtering
page.getByRole('listitem').filter({ hasText: 'Item 2' })
page.getByRole('listitem').filter({ has: page.getByText('Text') })

// Operators
page.getByRole('button').and(page.getByTitle('Submit'))
page.getByRole('button').or(page.getByTitle('Send'))

// Position (use carefully)
page.getByRole('button').first()
page.getByRole('button').last()
page.getByRole('button').nth(2)
```

## Related Resources

- [Other Locators Guide](https://playwright.dev/docs/other-locators)
- [Actionability](https://playwright.dev/docs/actionability)
- [Code Generator](https://playwright.dev/docs/codegen)
- [API Reference - Locator](https://playwright.dev/docs/api/class-locator)
- [W3C ARIA Roles](https://www.w3.org/TR/wai-aria-1.2/#roles)
