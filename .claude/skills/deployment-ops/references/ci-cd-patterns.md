# CI/CD Patterns

## The Principle

**Automate everything that can be automated, keep humans in the loop for decisions.**

CI/CD should catch problems early, deploy reliably, and give teams confidence to ship frequently.

---

## CI/CD Pipeline Structure

### Basic Pipeline

```yaml
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Build  │────▶│  Test   │────▶│ Deploy  │────▶│ Verify  │
│         │     │         │     │ Staging │     │         │
└─────────┘     └─────────┘     └─────────┘     └─────────┘
                                      │
                                      ▼
                                ┌─────────┐
                                │ Deploy  │
                                │  Prod   │
                                └─────────┘
```

### Advanced Pipeline

```yaml
┌─────────────────────────────────────────────────────────────────┐
│                         On Push/PR                               │
├─────────┬─────────┬─────────┬─────────┬─────────────────────────┤
│  Lint   │  Build  │  Unit   │ Integ.  │      Security           │
│         │         │  Tests  │  Tests  │       Scan              │
└────┬────┴────┬────┴────┬────┴────┬────┴──────────┬──────────────┘
     │         │         │         │               │
     └─────────┴─────────┴─────────┴───────────────┘
                         │
                    All Pass?
                         │
              ┌──────────┴──────────┐
              │                     │
         PR Merged            PR Open
              │                     │
              ▼                     ▼
        ┌─────────┐          ┌─────────┐
        │ Deploy  │          │ Preview │
        │ Staging │          │  Env    │
        └────┬────┘          └─────────┘
             │
        ┌────┴────┐
        │  E2E    │
        │  Tests  │
        └────┬────┘
             │
        Manual Gate
             │
        ┌────┴────┐
        │ Deploy  │
        │  Prod   │
        └────┬────┘
             │
        ┌────┴────┐
        │ Smoke   │
        │ Tests   │
        └─────────┘
```

---

## GitHub Actions Examples

### Basic CI

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Lint
        run: npm run lint

      - name: Type check
        run: npm run typecheck

      - name: Test
        run: npm test

      - name: Build
        run: npm run build
```

### Parallel Jobs

```yaml
name: CI

on: [push, pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "npm" }
      - run: npm ci
      - run: npm run lint

  typecheck:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "npm" }
      - run: npm ci
      - run: npm run typecheck

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "npm" }
      - run: npm ci
      - run: npm test

  build:
    runs-on: ubuntu-latest
    needs: [lint, typecheck, test]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "npm" }
      - run: npm ci
      - run: npm run build
      - uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
```

### Deploy Pipeline

```yaml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    outputs:
      version: ${{ steps.version.outputs.version }}
    steps:
      - uses: actions/checkout@v4

      - name: Get version
        id: version
        run: echo "version=$(node -p "require('./package.json').version")" >> $GITHUB_OUTPUT

      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "npm" }

      - run: npm ci
      - run: npm run build

      - uses: actions/upload-artifact@v4
        with:
          name: build-${{ steps.version.outputs.version }}
          path: dist/

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.build.outputs.version }}
          path: dist/

      - name: Deploy to Staging
        run: |
          # Deploy command here
          echo "Deploying to staging..."

  e2e-tests:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - name: Run E2E tests
        run: npm run test:e2e
        env:
          TEST_URL: https://staging.example.com

  deploy-production:
    needs: e2e-tests
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: build-${{ needs.build.outputs.version }}
          path: dist/

      - name: Deploy to Production
        run: |
          echo "Deploying to production..."

      - name: Smoke tests
        run: |
          curl -f https://example.com/health || exit 1
```

### Release Workflow

```yaml
name: Release

on:
  push:
    tags:
      - "v*"

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-node@v4
        with: { node-version: "20", cache: "npm" }

      - run: npm ci
      - run: npm run build

      - name: Generate changelog
        id: changelog
        run: |
          npx conventional-changelog -p angular -r 2 > RELEASE_NOTES.md

      - name: Create Release
        uses: softprops/action-gh-release@v1
        with:
          body_path: RELEASE_NOTES.md
          files: |
            dist/**
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Testing Strategies

### Test Pyramid in CI

```yaml
jobs:
  # Fast - run on every push
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:unit

  # Medium - run on every push
  integration-tests:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run test:integration
        env:
          DATABASE_URL: postgres://postgres:test@localhost:5432/test

  # Slow - run on main only
  e2e-tests:
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npx playwright install
      - run: npm run test:e2e
```

### Test Sharding

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        shard: [1, 2, 3, 4]
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test -- --shard=${{ matrix.shard }}/${{ strategy.job-total }}
```

---

## Environment Management

### Environment Protection

```yaml
# In GitHub repo settings, configure:
# - staging: auto-deploy
# - production: require approval

jobs:
  deploy-staging:
    environment: staging
    # Deploys automatically

  deploy-production:
    environment: production
    # Requires manual approval
```

### Environment Variables

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Deploy
        run: ./deploy.sh
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          ENVIRONMENT: production
```

---

## Deployment Patterns

### Blue-Green Deployment

```yaml
jobs:
  deploy:
    steps:
      - name: Deploy to Green
        run: |
          kubectl apply -f k8s/green/
          kubectl rollout status deployment/app-green

      - name: Run smoke tests
        run: |
          curl -f https://green.example.com/health

      - name: Switch traffic
        run: |
          kubectl patch service app -p '{"spec":{"selector":{"version":"green"}}}'

      - name: Verify
        run: |
          curl -f https://example.com/health
```

### Canary Deployment

```yaml
jobs:
  deploy:
    steps:
      - name: Deploy canary (10%)
        run: |
          kubectl apply -f k8s/canary/
          kubectl scale deployment app-canary --replicas=1

      - name: Monitor (5 minutes)
        run: |
          sleep 300
          ./scripts/check-metrics.sh

      - name: Promote to 50%
        run: |
          kubectl scale deployment app-canary --replicas=5

      - name: Monitor (5 minutes)
        run: |
          sleep 300
          ./scripts/check-metrics.sh

      - name: Full rollout
        run: |
          kubectl apply -f k8s/production/
          kubectl delete deployment app-canary
```

### Rolling Deployment

```yaml
# Kubernetes handles this automatically with:
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
```

---

## Security Scanning

### Dependency Scanning

```yaml
jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

### Code Scanning

```yaml
jobs:
  codeql:
    runs-on: ubuntu-latest
    permissions:
      security-events: write
    steps:
      - uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: javascript

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v2
```

---

## Notifications

### Slack Notifications

```yaml
jobs:
  deploy:
    steps:
      - name: Deploy
        run: ./deploy.sh

      - name: Notify success
        if: success()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Deployed ${{ github.sha }} to production"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}

      - name: Notify failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "❌ Deploy failed for ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
```

---

## Best Practices

### Speed

```yaml
# Cache dependencies
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}

# Run jobs in parallel
jobs:
  lint:
    ...
  test:
    ...
  build:
    needs: [lint, test]  # Only wait when necessary
```

### Reliability

```yaml
# Retry flaky steps
- name: Run tests
  uses: nick-invision/retry@v2
  with:
    timeout_minutes: 10
    max_attempts: 3
    command: npm test

# Set timeouts
jobs:
  build:
    timeout-minutes: 15
```

### Security

```yaml
# Minimal permissions
permissions:
  contents: read
  packages: write

# Pin action versions
- uses: actions/checkout@v4.1.0  # Not @v4 or @main

# Use environments for secrets
environment: production
```

---

## Summary

Effective CI/CD:

1. **Automate tests** — Unit, integration, E2E
2. **Parallelize** — Run independent jobs concurrently
3. **Use environments** — Staging before production
4. **Add gates** — Manual approval for production
5. **Monitor deployments** — Smoke tests, health checks
6. **Notify** — Success and failure alerts
7. **Secure** — Scan dependencies, minimal permissions
