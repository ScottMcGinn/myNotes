# Playwright CI/CD Setup Guide

This guide covers setting up Continuous Integration and Continuous Deployment (CI/CD) for Playwright tests across different platforms.

## Prerequisites

- Playwright project initialized
- Git repository set up
- Tests written and working locally

## Quick Start - Local Setup

### 1. Install Playwright
```powershell
npm init playwright@latest
```

### 2. Install Playwright Browsers
```powershell
npx playwright install
```

### 3. Run Tests Locally
```powershell
# Run all tests
npx playwright test

# Run in UI mode
npx playwright test --ui

# Run specific test file
npx playwright test tests/example.spec.ts

# Run in headed mode
npx playwright test --headed
```

## GitHub Actions CI/CD

### Basic GitHub Actions Workflow

Create `.github/workflows/playwright.yml`:

```yaml
name: Playwright Tests

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: lts/*
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    
    - name: Run Playwright tests
      run: npx playwright test
    
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 30
```

### Advanced GitHub Actions Configuration

```yaml
name: Playwright Tests (Advanced)

on:
  push:
    branches: [ main, master ]
  pull_request:
    branches: [ main, master ]
  schedule:
    # Run tests every day at 2 AM UTC
    - cron: '0 2 * * *'

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        shardIndex: [1, 2, 3, 4]
        shardTotal: [4]
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: 18
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps
    
    - name: Run Playwright tests
      run: npx playwright test --shard=${{ matrix.shardIndex }}/${{ matrix.shardTotal }}
      env:
        CI: true
    
    - name: Upload blob report to GitHub Actions Artifacts
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: blob-report-${{ matrix.shardIndex }}
        path: blob-report
        retention-days: 1
  
  merge-reports:
    # Merge reports after all tests complete
    if: always()
    needs: [test]
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: 18
        cache: 'npm'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Download blob reports from GitHub Actions Artifacts
      uses: actions/download-artifact@v4
      with:
        path: all-blob-reports
        pattern: blob-report-*
        merge-multiple: true
    
    - name: Merge into HTML Report
      run: npx playwright merge-reports --reporter html ./all-blob-reports
    
    - name: Upload HTML report
      uses: actions/upload-artifact@v4
      with:
        name: html-report--attempt-${{ github.run_attempt }}
        path: playwright-report
        retention-days: 14
```

### GitHub Actions with Multiple Browsers

```yaml
name: Cross-Browser Tests

on:
  push:
    branches: [ main ]

jobs:
  test:
    timeout-minutes: 60
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        browser: [chromium, firefox, webkit]
    steps:
    - uses: actions/checkout@v4
    
    - uses: actions/setup-node@v4
      with:
        node-version: lts/*
    
    - name: Install dependencies
      run: npm ci
    
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps ${{ matrix.browser }}
    
    - name: Run Playwright tests on ${{ matrix.browser }}
      run: npx playwright test --project=${{ matrix.browser }}
    
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report-${{ matrix.browser }}
        path: playwright-report/
        retention-days: 30
```

## Azure Pipelines CI/CD

### Basic Azure Pipeline

Create `azure-pipelines.yml`:

```yaml
trigger:
  - main
  - master

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: NodeTool@0
  inputs:
    versionSpec: '18.x'
  displayName: 'Install Node.js'

- script: npm ci
  displayName: 'Install dependencies'

- script: npx playwright install --with-deps
  displayName: 'Install Playwright browsers'

- script: npx playwright test
  displayName: 'Run Playwright tests'
  continueOnError: true

- task: PublishTestResults@2
  displayName: 'Publish test results'
  inputs:
    testResultsFormat: 'JUnit'
    testResultsFiles: 'test-results/junit.xml'
    failTaskOnFailedTests: true
  condition: always()

- task: PublishPipelineArtifact@1
  inputs:
    targetPath: playwright-report
    artifact: 'playwright-report'
    publishLocation: 'pipeline'
  displayName: 'Publish HTML report'
  condition: always()
```

### Advanced Azure Pipeline

```yaml
trigger:
  - main
  - master

pr:
  - main
  - master

pool:
  vmImage: 'ubuntu-latest'

variables:
  - name: CI
    value: true

stages:
- stage: Test
  jobs:
  - job: PlaywrightTests
    strategy:
      parallel: 4
    steps:
    - task: NodeTool@0
      inputs:
        versionSpec: '18.x'
      displayName: 'Install Node.js'
    
    - script: npm ci
      displayName: 'Install dependencies'
    
    - script: npx playwright install --with-deps
      displayName: 'Install Playwright browsers'
    
    - script: npx playwright test --shard=$(System.JobPositionInPhase)/$(System.TotalJobsInPhase)
      displayName: 'Run Playwright tests (Shard $(System.JobPositionInPhase))'
      continueOnError: true
    
    - task: PublishTestResults@2
      displayName: 'Publish test results'
      inputs:
        testResultsFormat: 'JUnit'
        testResultsFiles: 'test-results/junit.xml'
        mergeTestResults: true
        failTaskOnFailedTests: true
      condition: always()
    
    - task: PublishPipelineArtifact@1
      inputs:
        targetPath: playwright-report
        artifact: 'playwright-report-$(System.JobPositionInPhase)'
        publishLocation: 'pipeline'
      displayName: 'Publish HTML report'
      condition: always()
```

## GitLab CI/CD

### Basic GitLab CI

Create `.gitlab-ci.yml`:

```yaml
image: mcr.microsoft.com/playwright:v1.40.0-jammy

stages:
  - test

playwright-tests:
  stage: test
  script:
    - npm ci
    - npx playwright test
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 30 days
  retry:
    max: 2
    when:
      - runner_system_failure
      - stuck_or_timeout_failure
```

### Advanced GitLab CI

```yaml
image: mcr.microsoft.com/playwright:v1.40.0-jammy

stages:
  - install
  - test
  - report

variables:
  CI: "true"

cache:
  key:
    files:
      - package-lock.json
  paths:
    - node_modules/

install-dependencies:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 hour

playwright-tests:
  stage: test
  parallel: 4
  script:
    - npx playwright test --shard=$CI_NODE_INDEX/$CI_NODE_TOTAL
  artifacts:
    when: always
    paths:
      - test-results/
      - playwright-report/
    reports:
      junit: test-results/junit.xml
    expire_in: 1 day
  retry:
    max: 2

merge-reports:
  stage: report
  when: always
  dependencies:
    - playwright-tests
  script:
    - npx playwright merge-reports --reporter html ./test-results
  artifacts:
    when: always
    paths:
      - playwright-report/
    expire_in: 30 days
```

## Jenkins Pipeline

### Basic Jenkinsfile

Create `Jenkinsfile`:

```groovy
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.40.0-jammy'
        }
    }
    
    stages {
        stage('Install') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npx playwright test'
            }
        }
    }
    
    post {
        always {
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright Test Report'
            ])
        }
    }
}
```

### Advanced Jenkinsfile

```groovy
pipeline {
    agent {
        docker {
            image 'mcr.microsoft.com/playwright:v1.40.0-jammy'
        }
    }
    
    environment {
        CI = 'true'
    }
    
    parameters {
        choice(
            name: 'BROWSER',
            choices: ['chromium', 'firefox', 'webkit', 'all'],
            description: 'Browser to test'
        )
    }
    
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Chromium') {
                    when {
                        expression { params.BROWSER == 'chromium' || params.BROWSER == 'all' }
                    }
                    steps {
                        sh 'npx playwright test --project=chromium'
                    }
                }
                stage('Firefox') {
                    when {
                        expression { params.BROWSER == 'firefox' || params.BROWSER == 'all' }
                    }
                    steps {
                        sh 'npx playwright test --project=firefox'
                    }
                }
                stage('WebKit') {
                    when {
                        expression { params.BROWSER == 'webkit' || params.BROWSER == 'all' }
                    }
                    steps {
                        sh 'npx playwright test --project=webkit'
                    }
                }
            }
        }
    }
    
    post {
        always {
            junit testResults: 'test-results/junit.xml', allowEmptyResults: true
            
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: 'Playwright Test Results'
            ])
            
            archiveArtifacts artifacts: 'playwright-report/**/*', allowEmptyArchive: true
        }
    }
}
```

## Playwright Configuration for CI

### playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  
  // Run tests in files in parallel
  fullyParallel: true,
  
  // Fail the build on CI if you accidentally left test.only in the source code
  forbidOnly: !!process.env.CI,
  
  // Retry on CI only
  retries: process.env.CI ? 2 : 0,
  
  // Opt out of parallel tests on CI
  workers: process.env.CI ? 1 : undefined,
  
  // Reporter to use
  reporter: process.env.CI 
    ? [
        ['html'],
        ['junit', { outputFile: 'test-results/junit.xml' }],
        ['github']
      ]
    : 'html',
  
  // Shared settings for all projects
  use: {
    // Base URL to use in actions like `await page.goto('/')`
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    
    // Collect trace when retrying the failed test
    trace: 'on-first-retry',
    
    // Screenshot on failure
    screenshot: 'only-on-failure',
    
    // Video on failure
    video: 'retain-on-failure',
  },

  // Configure projects for major browsers
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
    // Mobile viewports
    {
      name: 'Mobile Chrome',
      use: { ...devices['Pixel 5'] },
    },
    {
      name: 'Mobile Safari',
      use: { ...devices['iPhone 12'] },
    },
  ],

  // Run your local dev server before starting the tests
  webServer: process.env.CI ? undefined : {
    command: 'npm run start',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

## Environment Variables

### Common Environment Variables for CI

```yaml
# In GitHub Actions
env:
  CI: true
  BASE_URL: https://staging.example.com
  API_KEY: ${{ secrets.API_KEY }}
  TEST_USER: ${{ secrets.TEST_USER }}
  TEST_PASSWORD: ${{ secrets.TEST_PASSWORD }}
```

### Using Secrets in Tests

```typescript
// tests/example.spec.ts
import { test, expect } from '@playwright/test';

test('login test', async ({ page }) => {
  await page.goto('/login');
  
  // Use environment variables
  await page.fill('#username', process.env.TEST_USER!);
  await page.fill('#password', process.env.TEST_PASSWORD!);
  
  await page.click('button[type="submit"]');
  
  await expect(page).toHaveURL('/dashboard');
});
```

## Docker Setup

### Dockerfile for Playwright

```dockerfile
FROM mcr.microsoft.com/playwright:v1.40.0-jammy

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

CMD ["npx", "playwright", "test"]
```

### docker-compose.yml

```yaml
version: '3.8'

services:
  playwright:
    build: .
    environment:
      - CI=true
      - BASE_URL=http://web:3000
    volumes:
      - ./playwright-report:/app/playwright-report
      - ./test-results:/app/test-results
    depends_on:
      - web
  
  web:
    image: nginx:alpine
    ports:
      - "3000:80"
    volumes:
      - ./public:/usr/share/nginx/html
```

## Best Practices for CI/CD

### 1. Optimize Test Execution Time

```typescript
// Use parallel execution
export default defineConfig({
  workers: process.env.CI ? 4 : undefined,
  fullyParallel: true,
});
```

### 2. Implement Test Sharding

```powershell
# Split tests across multiple jobs
npx playwright test --shard=1/4
npx playwright test --shard=2/4
npx playwright test --shard=3/4
npx playwright test --shard=4/4
```

### 3. Cache Dependencies

```yaml
# GitHub Actions caching
- uses: actions/setup-node@v4
  with:
    node-version: 18
    cache: 'npm'
```

### 4. Retry Failed Tests

```typescript
export default defineConfig({
  retries: process.env.CI ? 2 : 0,
});
```

### 5. Collect Artifacts

Always save:
- HTML reports
- Screenshots
- Videos
- Trace files
- Test results (JUnit XML)

### 6. Use Docker Images

```yaml
# Use official Playwright Docker image
jobs:
  test:
    runs-on: ubuntu-latest
    container:
      image: mcr.microsoft.com/playwright:v1.40.0-jammy
```

### 7. Set Timeouts

```typescript
export default defineConfig({
  timeout: 30000, // 30 seconds per test
  expect: {
    timeout: 5000, // 5 seconds for assertions
  },
});
```

### 8. Handle Flaky Tests

```typescript
test.describe('flaky test', () => {
  test.beforeEach(async ({ page }) => {
    // Add wait for network idle
    await page.goto('/', { waitUntil: 'networkidle' });
  });
  
  test('should be stable', async ({ page }) => {
    // Use waitFor instead of hard sleeps
    await page.waitForSelector('.element', { state: 'visible' });
  });
});
```

## Monitoring and Reporting

### 1. Integrate with Reporting Tools

```yaml
# Send results to external service
- name: Upload to Dashboard
  if: always()
  run: |
    npx playwright merge-reports --reporter=github ./blob-reports
```

### 2. Slack Notifications

```yaml
# GitHub Actions Slack notification
- name: Notify Slack
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    payload: |
      {
        "text": "Playwright tests failed on ${{ github.ref }}"
      }
```

### 3. Email Notifications

Configure in your CI platform settings to send emails on test failures.

## Troubleshooting CI/CD Issues

### Issue: Tests Pass Locally but Fail in CI

**Solutions:**
- Check environment variables
- Ensure same Node version
- Use `--headed` mode locally to simulate CI
- Check for timing issues
- Verify network conditions

### Issue: Slow Test Execution

**Solutions:**
- Implement test sharding
- Increase parallel workers
- Cache dependencies
- Use faster CI runners
- Optimize selectors

### Issue: Flaky Tests

**Solutions:**
- Add proper waits
- Use `waitForLoadState('networkidle')`
- Avoid hard timeouts
- Use auto-retry
- Check for race conditions

## Additional Resources

- [Playwright CI Documentation](https://playwright.dev/docs/ci)
- [GitHub Actions Playwright](https://playwright.dev/docs/ci-intro)
- [Docker Hub - Playwright](https://hub.docker.com/_/microsoft-playwright)
- [Playwright Best Practices](https://playwright.dev/docs/best-practices)
