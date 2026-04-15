# DIY Accounting Submit - GitHub Copilot Code Review Instructions

**Last Updated:** 2026-01-05

## About This File

This file contains guidelines for **GitHub Copilot** code review agent. The repository also has guidelines for other AI coding assistants:
- `CLAUDE.md` + `.claude/rules/` - Guidelines for Claude Code (emphasis on autonomous task execution & implementation)
- `.junie/guidelines.md` - Guidelines for Junie (custom agent, emphasis on testing & iteration)

Each assistant has complementary strengths - GitHub Copilot is optimized for code review, analysis, and providing thoughtful feedback.

## Purpose

These instructions guide GitHub Copilot's code review agent to provide specialized, high-quality reviews for this repository. The focus is on **analysis and understanding** rather than test execution.

## Repository Documentation

**Primary Reference**: See [`./REPORT_REPOSITORY_CONTENTS.md`](../REPORT_REPOSITORY_CONTENTS.md) for comprehensive technical documentation including:

- Complete architecture overview (AWS serverless stack)
- All npm scripts in `package.json` with detailed descriptions
- Maven/CDK build process and stack organization
- Environment configuration (`.env.test`, `.env.ci`, `.env.proxy`, `.env.prod`)
- Testing strategy (unit, system, browser, behaviour)
- GitHub Actions CI/CD workflows
- Directory structure and file purposes
- AWS deployment architecture and security model

**When reviewing code**, reference REPORT_REPOSITORY_CONTENTS.md to understand context, verify script usage, and check consistency with documented patterns.

## Code Review Philosophy

### Favor Analysis Over Execution

As a code review agent, prioritize **static analysis and code comprehension** over running tests:

1. **Read and understand** code paths by tracing through files
2. **Mentally dry-run** logic to identify potential issues
3. **Validate consistency** with existing patterns and conventions
4. **Check references** against documented scripts and configuration
5. **Suggest tests** when appropriate, but let developers/CI run them

**Note**: The `.junie/guidelines.md` file describes behavior for the Junie custom agent, which emphasizes continuous
testing and iteration. As a code review agent, your role is complementary - you provide thoughtful analysis while Junie
handles execution and testing.

### Analysis Workflow

When reviewing code changes be as low friction as possible, maintaining current standards for the incoming code.

1. **Understand the context**
   - What problem is being solved?
   - Which components/files are affected?
   - What are the environmental implications?

2. **Trace code paths**
   - Follow execution flow through functions/modules
   - Identify dependencies and side effects
   - Check error handling and edge cases

3. **Validate against patterns**
   - Does it match existing code style?
   - Are naming conventions consistent?
   - Is it using documented npm scripts correctly?

4. **Consider cross-cutting concerns**
   - Security implications (secrets, IAM, input validation)
   - Performance and cost (Lambda execution, DynamoDB queries)
   - Testing coverage (are appropriate tests included?)
   - Environmental differences (test vs. CI vs. prod)

5. **Suggest minimal changes**
   - Focus on surgical, targeted improvements
   - Preserve working code unless fixing security issues
   - Match local style over enforcing global style rules
   - Do not raise issues where new possibly problematic code has been introduced but this pattern is already established
   - Do not raise issues about unused imports or variables or formatting
   - Do do raise concerns about performance where the suggested optimisation is unlikely to be a measurable signal above the noise.

## Repository Patterns and Conventions

### Code Style and Formatting

**JavaScript/TypeScript** (ES Modules):
- **Linter**: ESLint with flat config (`eslint.config.js`)
- **Formatter**: Prettier (`.prettierrc`)
- **Scripts**: Only run if specifically asked to fix formatting and linting errors: `npm run linting`, `npm run linting-fix`, `npm run formatting`, `npm run formatting-fix`
- **Convention**: Only run if specifically asked to fix formatting and linting errors: `npm run linting-fix && npm run formatting-fix`

**Java** (AWS CDK Infrastructure):
- **Formatter**: Spotless with Palantir Java Format (100-column width)
- **Scripts**: Only run if specifically asked to fix formatting and linting errors: `./mvnw spotless:check`, `./mvnw spotless:apply`
- **Convention**: Runs during Maven `install` phase, fails build if not formatted

**General Style Rule**: Match existing local style rather than forcing global rules when it would be disruptive. Only change style in code you're already modifying.

Avoid unnecessary formatting changes when editing code.
For the lines that you change, be compliant with the formatting rules.
Do not run formatting tools on the whole repository or whole files unless the whole file is new.

### Testing Strategy

**Test**: Run the following test commands in sequence to check that the code works:
```
npm test
./mvnw clean verify
npm run test:submitVatBehaviour-proxy
```
If you need to capture the output of a test do it like this:
```
npm test > target/test.txt 2>&1
./mvnw clean verify > target/mvnw.txt 2>&1
npm run test:submitVatBehaviour-proxy > target/behaviour.txt 2>&1
```
And query for a subset of things that might be of interest fail|error with:
```
grep -i -n -A 20 -E 'fail|error' target/test.txt
grep -i -n -A 20 -E 'fail|error' target/mvnw.txt
grep -i -n -A 20 -E 'fail|error' target/behaviour.txt
```

This repository uses a **four-tier testing pyramid**:

1. **Unit Tests** (Vitest): Fast, isolated tests of individual functions/modules
   - Location: `app/unit-tests/`, `web/unit-tests/`
   - Run: `npm run test:unit` (~4 seconds)
   - Focus: Business logic, helpers, utilities

2. **System Tests** (Vitest + Docker): Integration tests with real dependencies
   - Location: `app/system-tests/`
   - Run: `npm run test:system` (~6 seconds)
   - Focus: Service integration, Docker containers (DynamoDB, OAuth2)

3. **Browser Tests** (Playwright): UI component tests in real browser
   - Location: `web/browser-tests/`
   - Run: `npm run test:browser` (~30+ seconds)
   - Focus: Frontend widgets, navigation, client-side logic

4. **Behaviour Tests** (Playwright): End-to-end user journey tests
   - Location: `behaviour-tests/`
   - Run: `npm run test:allBehaviour` (with environment variants: `-proxy`, `-ci`, `-prod`)
   - Focus: Complete flows (auth, VAT submission, bundles, receipts)

**Default test command**: `npm test` runs unit + system tests (~4 seconds, 108 tests)

### Environment Configuration

The repository supports **four environments** via `.env` files:

| Environment | File | Purpose |
|------------|------|---------|
| **test** | `.env.test` | Unit/system tests with mocked services |
| **proxy** | `.env.proxy` | Local development with ngrok, mock OAuth2, local DynamoDB |
| **ci** | `.env.ci` | Continuous integration with real AWS resources |
| **prod** | `.env.prod` | Production deployment |

**Key environment variables** to be aware of:
- `ENVIRONMENT_NAME`: `test`, `ci`, or `prod`
- `DEPLOYMENT_NAME`: Unique deployment identifier
- `DIY_SUBMIT_BASE_URL`: Application base URL
- `HMRC_BASE_URI`: HMRC API endpoint (test or prod)
- `COGNITO_USER_POOL_ID`, `COGNITO_CLIENT_ID`: AWS Cognito configuration
- `*_DYNAMODB_TABLE_NAME`: DynamoDB table names
- `*_CLIENT_SECRET_ARN`: AWS Secrets Manager ARNs (never plain secrets in env files)

### AWS CDK Architecture

The infrastructure is divided into **two CDK applications**:

1. **Environment Stacks** (`cdk-environment/`): Long-lived, shared resources
   - ObservabilityStack (CloudWatch RUM, logs, alarms)
   - DataStack (DynamoDB tables)
   - ApexStack (Route53 DNS)
   - IdentityStack (Cognito user pool)
   - Deployed by: `deploy-environment.yml` workflow

2. **Application Stacks** (`cdk-application/`): Per-deployment resources
   - DevStack (S3, CloudFront, ECR)
   - AuthStack, HmrcStack, AccountStack (Lambda functions)
   - ApiStack (HTTP API Gateway)
   - EdgeStack (CloudFront distribution)
   - PublishStack (static file deployment)
   - OpsStack (monitoring dashboard)
   - SelfDestructStack (auto-cleanup for non-prod)
   - Deployed by: `deploy.yml` workflow

**Entry points**:
- `infra/main/java/co/uk/diyaccounting/submit/SubmitEnvironment.java`
- `infra/main/java/co/uk/diyaccounting/submit/SubmitApplication.java`

### Available npm Scripts

See REPORT_REPOSITORY_CONTENTS.md Section "Package.json Operations" for the complete reference of all npm scripts. Key scripts include:

**Build & Deploy**:
- `npm run build` - Full Maven build + restore deployment markers
- `npm start` - Start all local services (proxy, auth, data, server)
- `npm run server` - Start Express server on port 3000

**Testing**:
- `npm test` - Default: unit + system tests
- `npm run test:unit` - Unit tests only
- `npm run test:system` - System tests only
- `npm run test:browser` - Playwright browser tests
- `npm run test:allBehaviour` - End-to-end behaviour tests

**Code Quality**:
- `npm run formatting` - Check JS/Java formatting
- `npm run formatting-fix` - Auto-fix JS/Java formatting - Only run if specifically asked to fix formatting and linting errors:
- `npm run linting` - Check ESLint rules
- `npm run linting-fix` - Auto-fix ESLint issues - Only run if specifically asked to fix formatting and linting errors:

**Local Development**:
- `npm run proxy` - Start ngrok proxy
- `npm run auth` - Start mock OAuth2 server (Docker)
- `npm run data` - Start local DynamoDB (dynalite)

## Code Review Focus Areas

### Security Considerations

**Critical**: Always check for security issues in code changes:

1. **Secrets Management**
   - ❌ Never commit secrets to code or environment files
   - ✅ Use AWS Secrets Manager ARNs in `.env.ci` and `.env.prod`
   - ✅ Export secrets in shell for local development
   - Check: Search for patterns like `client_secret`, `password`, API keys

2. **IAM Permissions**
   - Check Lambda execution roles for least privilege
   - Verify CDK-created roles follow principle of minimal access
   - Look for overly broad wildcards in IAM policies (`Resource: "*"`)

3. **Input Validation**
   - Validate and sanitize user input in Lambda functions
   - Check for SQL injection risks (though DynamoDB SDK helps here)
   - Verify OAuth state parameter validation

4. **Authentication & Authorization**
   - Ensure protected routes use custom authorizer
   - Check JWT validation logic in `app/functions/auth/customAuthorizer.js`
   - Verify bundle entitlement checks before feature access

### Consistency with Patterns

**Naming Conventions**:
- Lambda function files: `{feature}{Method}.js` (e.g., `hmrcVatReturnPost.js`)
- CDK stacks: `{Purpose}Stack` (e.g., `AuthStack`, `HmrcStack`)
- DynamoDB tables: `{env}-submit-{purpose}` (e.g., `ci-submit-bundles`)
- Environment variables: `{SERVICE}_{RESOURCE}_ARN` format
- npm scripts: Use `:` separator for variants (e.g., `test:unit`, `test:submitVatBehaviour-proxy`)

**Error Handling**:
- Lambda functions should catch errors and return appropriate HTTP status codes
- Use structured logging with Pino logger
- Include correlation IDs for request tracing

**Testing Patterns**:
- Unit tests use Vitest with `happy-dom` for DOM testing
- System tests use `testcontainers` pattern with Docker
- Behaviour tests use Playwright with page object pattern
- Mock external APIs with MSW (Mock Service Worker)

### Environmental Impact

When reviewing changes, consider impact across **all four environments**:

1. **Test environment** (`.env.test`):
   - Uses mocked/stubbed services
   - Fast, isolated, no external dependencies
   - Safe to break temporarily during development

2. **Proxy environment** (`.env.proxy`):
   - Local development setup
   - Requires ngrok, Docker for OAuth2/DynamoDB
   - Changes should work locally for developers

3. **CI environment** (`.env.ci`):
   - Real AWS resources but short-lived
   - Automated testing in GitHub Actions
   - Self-destructs after 8 hours (SelfDestructStack)

4. **Production environment** (`.env.prod`):
   - Real users and data
   - High reliability and security requirements
   - Changes must be backwards compatible

**Red flags**:
- Hard-coded environment names or URLs
- Assumptions about resource existence (check environment stack deployment)
- Different behavior in test vs. prod due to missing mocks

### Testing Implications

When reviewing code changes, check:

1. **Are new tests included?**
   - New functions/features should have unit tests
   - New API endpoints should have system/behaviour tests
   - UI changes should have browser tests

2. **Do existing tests need updates?**
   - Check if mocks need to match new behavior
   - Update test data/fixtures if format changed
   - Verify test names still match what they test

3. **Is test coverage appropriate?**
   - Business logic: High unit test coverage expected
   - Integration points: System tests for happy path + errors
   - User journeys: Behaviour tests for critical flows only

4. **Are tests discoverable?**
   - Test files follow `*.test.js` naming convention
   - Located in appropriate directory (`unit-tests/`, `system-tests/`, etc.)
   - Can be run via documented npm scripts

### Performance and Cost Considerations

**AWS Lambda**:
- Cold start times (Node.js 22 runtime, Docker images from ECR)
- Memory allocation (default: check CDK stack definitions)
- Execution duration (DynamoDB queries, HMRC API calls)

**DynamoDB**:
- On-demand billing (no provisioned capacity)
- Query efficiency (use partition key + sort key)
- Item size limits (400 KB per item)

**CloudFront**:
- Cache policy configuration (static vs. dynamic content)
- Origin request optimization
- CloudWatch RUM costs (events per session)

**Cost red flags**:
- Unbounded loops or recursive calls in Lambda
- Full table scans on DynamoDB
- Excessive CloudWatch log volume
- 100% session sampling in RUM (consider lower rate for prod)

## Common Patterns to Recognize

### Lambda Function Pattern

```javascript
// Standard Lambda function structure
export const ingestHandler = async (event, context) => {
  try {
    // 1. Extract parameters from event (query, path, headers, body)
    // 2. Validate input
    // 3. Perform business logic
    // 4. Call AWS services (DynamoDB, Secrets Manager, etc.)
    // 5. Return successful response
    return {
      statusCode: 200,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(result)
    };
  } catch (error) {
    // 6. Log error and return appropriate status code
    logger.error({ error, event }, 'Lambda execution failed');
    return {
      statusCode: 500,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ error: 'Internal server error' })
    };
  }
};
```

### CDK Stack Pattern

```java
// Standard CDK stack structure
public class ExampleStack extends Stack {
    public ExampleStack(final Construct scope, final String id, final ExampleStackProps props) {
        super(scope, id, props);

        // 1. Create IAM roles
        // 2. Create Lambda functions (from Docker images)
        // 3. Create API Gateway routes
        // 4. Create CloudWatch alarms/dashboards
        // 5. Output important values (ARNs, URLs)
    }
}
```

### Environment Variable Pattern

```javascript
// Reading environment variables with fallbacks
const tableName = process.env.BUNDLE_DYNAMODB_TABLE_NAME || 'default-table';
const region = process.env.AWS_REGION || 'eu-west-2';

// For secrets, use AWS Secrets Manager
const secretArn = process.env.HMRC_CLIENT_SECRET_ARN;
if (!secretArn) {
  throw new Error('HMRC_CLIENT_SECRET_ARN is required');
}
```

## Recommended Review Checklist

When reviewing a pull request, work through these checks:

- [ ] **Read the PR description** - Understand the goal and context
- [ ] **Review changed files** - What components are affected?
- [ ] **Trace code paths** - Follow execution flow mentally
- [ ] **Check security** - Any secrets, IAM, or input validation issues?
- [ ] **Validate consistency** - Naming, patterns, error handling match existing code?
- [ ] **Verify scripts** - Any new/changed npm scripts documented correctly?
- [ ] **Consider environments** - Impact on test/proxy/ci/prod?
- [ ] **Check tests** - Are appropriate tests included/updated?
- [ ] **Review performance** - Any Lambda/DynamoDB/CloudFront concerns?
- [ ] **Validate references** - Do script names match `package.json`?
- [ ] **Check style** - Does formatting match repository conventions?
- [ ] **Suggest improvements** - Can code be simpler, clearer, more maintainable?

## Resources and References

- **Repository Documentation**: [`./REPORT_REPOSITORY_CONTENTS.md`](../REPORT_REPOSITORY_CONTENTS.md)
- **README**: [`./README.md`](../README.md)
- **Package Scripts**: [`./package.json`](../package.json)
- **Maven Build**: [`./pom.xml`](../pom.xml)
- **ESLint Config**: [`./eslint.config.js`](../eslint.config.js)
- **Vitest Config**: [`./vitest.config.js`](../vitest.config.js)
- **Playwright Config**: [`./playwright.config.js`](../playwright.config.js)
- **GitHub Workflows**: [`./.github/workflows/`](../workflows/)

---

**Remember**: Your role as a code review agent is to provide thoughtful, analytical feedback based on understanding the code and its context. Leave test execution to developers and CI systems. Focus on what you do best: pattern recognition, consistency checking, and architectural guidance.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/antonycc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
