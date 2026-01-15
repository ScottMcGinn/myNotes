# Playwright CLI Commands Reference

Complete reference guide for Playwright command-line interface commands.

## Installation & Setup

### Install Playwright
```powershell
# Install with npm
npm init playwright@latest

# Install specific version
npm install -D @playwright/test@1.40.0

# Install with Yarn
yarn create playwright

# Install with pnpm
pnpm create playwright
```

### Install Browsers
```powershell
# Install all browsers
npx playwright install

# Install specific browser
npx playwright install chromium
npx playwright install firefox
npx playwright install webkit

# Install browsers with system dependencies
npx playwright install --with-deps

# Install specific browser with dependencies
npx playwright install chromium --with-deps
```

### Update Playwright
```powershell
# Update to latest version
npm install -D @playwright/test@latest

# Update browsers after upgrading
npx playwright install
```

## Running Tests

### Basic Test Execution
```powershell
# Run all tests
npx playwright test

# Run tests in specific file
npx playwright test tests/example.spec.ts

# Run tests in specific folder
npx playwright test tests/auth/

# Run multiple files
npx playwright test tests/login.spec.ts tests/signup.spec.ts
```

### Test Filtering
```powershell
# Run tests by name pattern (grep)
npx playwright test -g "login"
npx playwright test --grep "authentication"

# Run tests NOT matching pattern
npx playwright test --grep-invert "slow"

# Run tests by tag
npx playwright test --grep "@smoke"

# Multiple grep patterns (AND logic)
npx playwright test --grep "login" --grep "admin"
```

### Browser Selection
```powershell
# Run in specific project/browser
npx playwright test --project=chromium
npx playwright test --project=firefox
npx playwright test --project=webkit

# Run in multiple browsers
npx playwright test --project=chromium --project=firefox

# List all configured projects
npx playwright show-config
```

### Test Modes
```powershell
# Run in headed mode (visible browser)
npx playwright test --headed

# Run in debug mode
npx playwright test --debug

# Run in UI mode (interactive)
npx playwright test --ui

# Run with trace
npx playwright test --trace on
npx playwright test --trace retain-on-failure

# Run with video
npx playwright test --video on
npx playwright test --video retain-on-failure

# Run with screenshots
npx playwright test --screenshot on
npx playwright test --screenshot only-on-failure
```

### Parallel Execution
```powershell
# Set number of workers
npx playwright test --workers=4
npx playwright test --workers=50%

# Run serially (one at a time)
npx playwright test --workers=1

# Fully parallel (each test in parallel)
npx playwright test --fully-parallel

# Shard tests across multiple machines
npx playwright test --shard=1/4  # Machine 1
npx playwright test --shard=2/4  # Machine 2
npx playwright test --shard=3/4  # Machine 3
npx playwright test --shard=4/4  # Machine 4
```

### Retry & Repeat
```powershell
# Set retry count
npx playwright test --retries=2

# Repeat each test multiple times
npx playwright test --repeat-each=3

# Run only last failed tests
npx playwright test --last-failed
```

### Output & Reporting
```powershell
# Use specific reporter
npx playwright test --reporter=html
npx playwright test --reporter=json
npx playwright test --reporter=junit
npx playwright test --reporter=list
npx playwright test --reporter=dot
npx playwright test --reporter=line
npx playwright test --reporter=github

# Multiple reporters
npx playwright test --reporter=html,junit

# Quiet mode (minimal output)
npx playwright test --quiet

# Verbose output
npx playwright test --reporter=list -v
```

### Timeout Configuration
```powershell
# Set global timeout (milliseconds)
npx playwright test --timeout=60000

# Set global timeout for each hook
npx playwright test --global-timeout=3600000

# Pass through timeout to test
npx playwright test -- --timeout=30000
```

### Other Test Options
```powershell
# Run tests in specific line
npx playwright test example.spec.ts:42

# Update snapshots
npx playwright test --update-snapshots
npx playwright test -u

# Ignore snapshots
npx playwright test --ignore-snapshots

# Set base URL
npx playwright test --base-url=https://staging.example.com

# Pass through custom config
npx playwright test --config=playwright.staging.config.ts

# Set maximum failures
npx playwright test --max-failures=5
npx playwright test -x  # Stop after first failure
```

## Code Generation

### Codegen - Generate Test Code
```powershell
# Basic code generation
npx playwright codegen

# Generate code for specific URL
npx playwright codegen https://example.com

# Generate code in specific browser
npx playwright codegen --browser=chromium
npx playwright codegen --browser=firefox
npx playwright codegen --browser=webkit

# Set viewport size
npx playwright codegen --viewport-size=1280,720

# Generate code with device emulation
npx playwright codegen --device="iPhone 13"
npx playwright codegen --device="Pixel 5"

# Set color scheme
npx playwright codegen --color-scheme=dark
npx playwright codegen --color-scheme=light

# Set geolocation
npx playwright codegen --geolocation="41.890221,12.492348"

# Set language
npx playwright codegen --lang=de-DE

# Set timezone
npx playwright codegen --timezone="Europe/Rome"

# Save output to file
npx playwright codegen --target=playwright-test -o tests/generated.spec.ts

# Generate in different languages
npx playwright codegen --target=playwright-test  # TypeScript (default)
npx playwright codegen --target=playwright       # JavaScript
npx playwright codegen --target=python
npx playwright codegen --target=python-pytest
npx playwright codegen --target=java
npx playwright codegen --target=csharp

# Load existing state
npx playwright codegen --load-storage=state.json
```

## Debugging & Inspection

### Debug Tests
```powershell
# Debug mode (step through)
npx playwright test --debug

# Debug specific test
npx playwright test example.spec.ts --debug

# Debug from specific line
npx playwright test example.spec.ts:42 --debug

# Playwright Inspector
PWDEBUG=1 npx playwright test

# Headed mode with slowMo
npx playwright test --headed --slow-mo=1000
```

### Trace Viewer
```powershell
# Show trace file
npx playwright show-trace trace.zip

# Show trace from test results
npx playwright show-trace test-results/example-test/trace.zip
```

### Show Report
```powershell
# Open last HTML report
npx playwright show-report

# Open specific report
npx playwright show-report playwright-report

# Serve report on specific port
npx playwright show-report --port=9323
```

## Screenshots & PDFs

### Screenshot Command
```powershell
# Take screenshot of a page
npx playwright screenshot https://example.com screenshot.png

# Full page screenshot
npx playwright screenshot --full-page https://example.com screenshot.png

# Set viewport
npx playwright screenshot --viewport-size=1280,720 https://example.com screenshot.png

# Use specific browser
npx playwright screenshot --browser=webkit https://example.com screenshot.png

# Set device
npx playwright screenshot --device="iPhone 13" https://example.com screenshot.png

# Wait for timeout before screenshot
npx playwright screenshot --wait-for-timeout=3000 https://example.com screenshot.png
```

### PDF Generation
```powershell
# Generate PDF
npx playwright pdf https://example.com page.pdf

# Set page size
npx playwright pdf --format=A4 https://example.com page.pdf
npx playwright pdf --format=Letter https://example.com page.pdf

# Set margins
npx playwright pdf --margin=10mm https://example.com page.pdf

# Landscape orientation
npx playwright pdf --landscape https://example.com page.pdf
```

## Browser Control

### List Browsers
```powershell
# List installed browsers
npx playwright list

# List all available browsers
npx playwright install --dry-run
```

### Open Browser
```powershell
# Open browser
npx playwright open https://example.com

# Open specific browser
npx playwright open --browser=chromium https://example.com
npx playwright open --browser=firefox https://example.com
npx playwright open --browser=webkit https://example.com

# Open with device emulation
npx playwright open --device="iPhone 13" https://example.com

# Open in specific color scheme
npx playwright open --color-scheme=dark https://example.com

# Open with custom viewport
npx playwright open --viewport-size=1280,720 https://example.com

# Open with saved storage state
npx playwright open --load-storage=state.json https://example.com

# Save storage state
npx playwright open --save-storage=state.json https://example.com
```

### Clear Cache
```powershell
# Uninstall browsers
npx playwright uninstall

# Uninstall specific browser
npx playwright uninstall chromium

# Uninstall and remove dependencies
npx playwright uninstall --all
```

## Configuration & Info

### Show Config
```powershell
# Show resolved configuration
npx playwright show-config

# Show specific project config
npx playwright show-config --project=chromium

# Show config in different format
npx playwright show-config --json
```

### Version Info
```powershell
# Show Playwright version
npx playwright --version

# Show detailed info
npx playwright --help
```

### Test List
```powershell
# List all tests
npx playwright test --list

# List tests matching pattern
npx playwright test --list --grep="login"

# List tests in specific project
npx playwright test --list --project=chromium
```

## Merge Reports

### Merge Blob Reports
```powershell
# Merge blob reports into HTML
npx playwright merge-reports ./blob-reports

# Merge with specific reporter
npx playwright merge-reports --reporter=html ./blob-reports

# Merge multiple reporters
npx playwright merge-reports --reporter=html,junit ./blob-reports

# Output to specific directory
npx playwright merge-reports --reporter=html ./blob-reports -o ./merged-report
```

## Server (Experimental)

### Test Server
```powershell
# Start test server
npx playwright test-server

# Start on specific port
npx playwright test-server --port=3000
```

## Environment Variables

### Set via Command Line
```powershell
# Windows PowerShell
$env:DEBUG="pw:api"; npx playwright test
$env:PWDEBUG="1"; npx playwright test
$env:PLAYWRIGHT_BROWSER="chromium"; npx playwright test

# Set base URL
$env:BASE_URL="https://staging.example.com"; npx playwright test

# Set browser executable
$env:PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH="C:\chrome\chrome.exe"; npx playwright test
```

### Common Environment Variables
```powershell
# Debug API calls
DEBUG=pw:api npx playwright test

# Debug Playwright Inspector
PWDEBUG=1 npx playwright test

# Browser executable paths
PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH=/path/to/chrome
PLAYWRIGHT_FIREFOX_EXECUTABLE_PATH=/path/to/firefox
PLAYWRIGHT_WEBKIT_EXECUTABLE_PATH=/path/to/webkit

# Skip browser downloads
PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1

# Set browser download directory
PLAYWRIGHT_BROWSERS_PATH=/path/to/browsers
```

## Helper Commands

### Install System Dependencies
```powershell
# Install system dependencies (Linux only)
npx playwright install-deps

# Install deps for specific browser
npx playwright install-deps chromium
```

### Check Installation
```powershell
# Verify Playwright is working
npx playwright --version

# Check Node.js version
node --version

# Check npm version
npm --version
```

## Quick Command Reference

### Most Used Commands
```powershell
# Initialize new project
npm init playwright@latest

# Install browsers
npx playwright install

# Run tests
npx playwright test

# Run in UI mode
npx playwright test --ui

# Debug tests
npx playwright test --debug

# Generate code
npx playwright codegen

# Show report
npx playwright show-report

# View trace
npx playwright show-trace trace.zip

# Run headed
npx playwright test --headed

# Run specific browser
npx playwright test --project=chromium

# Update snapshots
npx playwright test -u
```

## Combining Options

### Common Combinations
```powershell
# Run specific tests in debug mode
npx playwright test login.spec.ts --debug

# Run in headed mode with specific browser
npx playwright test --headed --project=chromium

# Run with retries and HTML report
npx playwright test --retries=2 --reporter=html

# Run specific grep pattern in specific browser
npx playwright test --grep="smoke" --project=firefox

# Run in UI mode for specific file
npx playwright test login.spec.ts --ui

# Generate code with device emulation and save to file
npx playwright codegen --device="iPhone 13" -o tests/mobile.spec.ts https://example.com

# Run tests with trace on failure
npx playwright test --trace=retain-on-failure --reporter=html

# Shard tests with workers
npx playwright test --shard=1/4 --workers=2

# Run last failed tests in debug mode
npx playwright test --last-failed --debug
```

## Getting Help

### Help Commands
```powershell
# General help
npx playwright --help

# Help for specific command
npx playwright test --help
npx playwright codegen --help
npx playwright install --help
npx playwright show-trace --help

# List all available commands
npx playwright
```

## Platform-Specific Notes

### Windows PowerShell
```powershell
# Set environment variable
$env:VARIABLE="value"

# Run command
npx playwright test

# Clear environment variable
Remove-Item Env:\VARIABLE
```

### Windows CMD
```cmd
# Set environment variable
set VARIABLE=value

# Run command
npx playwright test
```

### Linux/Mac
```bash
# Set environment variable
export VARIABLE=value

# Run command
npx playwright test

# One-liner
VARIABLE=value npx playwright test
```

## Tips & Tricks

1. **Use --ui for development** - Interactive mode is great for debugging
2. **Use --headed for visual debugging** - See what's happening
3. **Use --grep for quick tests** - Run specific test sets
4. **Use sharding for CI** - Speed up test execution
5. **Use --debug for step-through** - Inspect test execution
6. **Use codegen to learn selectors** - Great for finding locators
7. **Use --update-snapshots carefully** - Review changes before committing
8. **Use --reporter=html for detailed results** - Best for local review
9. **Use --workers=1 for flaky tests** - Helps debug timing issues
10. **Use --project for cross-browser** - Test in multiple browsers

## Resources

- [Official CLI Documentation](https://playwright.dev/docs/test-cli)
- [Configuration File Reference](https://playwright.dev/docs/test-configuration)
- [Test Runner API](https://playwright.dev/docs/api/class-test)
- [Environment Variables](https://playwright.dev/docs/test-advanced#launching-a-development-web-server-during-the-tests)
