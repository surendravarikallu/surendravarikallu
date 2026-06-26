# Skillnox Testing Documentation

This document describes the testing architecture, configurations, test runner commands, and report inspection steps for the Skillnox Contest Platform.

---

## 1. Testing Architecture Overview

Skillnox uses a multi-layered testing strategy to guarantee software quality:

1. **Unit Testing (Jest + React Testing Library)**:
   Tests individual helper functions, utilities, and isolated components.
   - **Target**: `client/src/lib/contestUtils.ts` (contest status helpers) and `client/src/components/contest-timer.tsx` (the timer logic).
2. **Integration Testing (Jest + React Testing Library)**:
   Tests rendering, sorting, and user-facing conditions for complex UI components interacting with mock hooks.
   - **Target**: `client/src/components/leaderboard.tsx` (live leaderboard ranking, user highlights, and DQ badges).
3. **End-to-End (E2E) Testing (Playwright)**:
   Simulates full user flows in a real Chromium browser, checking authentication, redirection, and visual displays against the active local database and server.
   - **Target**: `tests/e2e/auth.spec.ts` (authentication failure and success).

---

## 2. Test Execution Commands

The following command scripts are configured in `package.json` for running tests:

| Command | Action | Description |
|---------|--------|-------------|
| `npm run test` | Run Jest Tests | Runs all unit and integration tests under JSDOM environment. |
| `npm run test:coverage` | Run Jest Coverage | Runs Jest tests and outputs a line-by-line coverage report. |
| `npm run test:e2e` | Run Playwright Tests | Launches Playwright E2E tests. Auto-starts local server. |
| `npm run test:e2e:report` | View E2E HTML Report | Opens the browser-based Playwright report dashboard. |

---

## 3. Configuration Details

### Unit & Integration Configuration (`jest.config.cjs`)
- **Preset**: `ts-jest` for native TypeScript support.
- **Environment**: `jest-environment-jsdom` to simulate a browser DOM environment.
- **Setup File**: `jest.setup.ts` imports `@testing-library/jest-dom` and mocks `framer-motion` to bypass JSDOM animation issues.
- **Compiler Config**: `tsconfig.jest.json` extends the main TSConfig to output CommonJS modules for Jest runtime.

### End-to-End Configuration (`playwright.config.ts`)
- **Test Directory**: `tests/e2e`.
- **Target Browser**: Chromium (Desktop Chrome).
- **Auto Server Start**: Configured to run `npm run dev` internally and verify that the app responds at `http://localhost:3000` before starting tests.
- **Reporter**: Outputs a fully interactive standalone HTML report.

---

## 4. Viewing Test Reports

### Jest Coverage Report
Running `npm run test:coverage` generates a `coverage/` folder in the project root.
- Open `coverage/lcov-report/index.html` in any web browser to view interactive visual code-coverage maps.

### Playwright E2E HTML Report
Running `npm run test:e2e` compiles results and creates a `playwright-report/` directory.
- Run `npm run test:e2e:report` to launch a local web server and open the browser interface displaying each test step, durations, and any screenshots or logs captured during failures.

---

## 5. Automated CI/CD Pipeline (GitHub Actions)

The platform runs a unified production-grade CI pipeline defined in `.github/workflows/ci-cd.yml` on every branch push and pull request:

1. **Continuous Integration (CI)**:
   - **Service Containers**: Starts isolated `PostgreSQL 15` and `Redis 6` containers to back the test environment.
   - **Dependency Installation**: Runs `npm ci` with cached dependencies.
   - **Type Checking**: Runs `npm run check` to ensure clean TypeScript compilation.
   - **Application Build**: Runs `npm run build` to verify frontend and backend build pipelines.
   - **Database Setup**: Executes database schema migration (`npm run db:migrate`) and seeds an admin user for authentication testing.
   - **Unit & Integration Tests**: Runs Jest test suites.
   - **Playwright E2E Tests**: Installs browser dependencies and runs full-browser automation tests.
   - **Artifact Collection**: Uploads the interactive Playwright HTML report as a workflow artifact upon E2E test failures.
