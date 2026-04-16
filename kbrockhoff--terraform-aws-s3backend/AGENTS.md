# AI Agent Git Ops – Brockhoff Repos

## Principles

* **Simplicity:** One clear purpose per operation.
* **Composable:** Steps build independently; easy to debug.

## Branch Management

* **Name:** `feature/descriptive-name`, `bugfix/issue-description` (kebab-case, branch from `main`).
* **Stay current:** `make update-branch`; resolve conflicts immediately.

## Commits

* **Pre-commit:** `make pre-commit` if not pushing, `make validate` if pushing.
* **Format:** [Conventional Commits](https://www.conventionalcommits.org/)
  `type(scope): description`
* **Scope:** Atomic, working-state, clear intent (“why”).

## Pull Requests

* **Pre-PR:** `make validate`
* **Template:** Use `.github/pull_request_template.md`

## Quality Gates

* **Automated:** `make validate` (must pass all: format, check, lint, test, docs)
* **Manual review:** Focus on simplicity, decoupling, clarity, and testability

## Troubleshooting

* **Rebase/merge conflicts:** Resolve immediately
* **Test/lint errors:** Fix root cause, don’t mask
* **Reset workspace:**
  `cd $(git rev-parse --show-toplevel) && pwd`
  `git reset --hard origin/main`
  `git clean -fd`
  `make clean`

## AI Agent Commands

* **Validate:** `make validate`
* **Quick check:** `make pre-commit`
* **Info:** `make status`
* **Resource count:** `make resource-count`
* **Cleanup:** `make clean`

**Best Practices:**

* Use single-command Make targets; Fail fast (`make validate`); keep context (`make status`); know rollback (`git reset`, `make clean`).

---

**Tip:** Prefer `make <target>` whenever possible.

---
Source: .ruler/instructions.md
---
# AWS Terraform S3 Backend Terraform Module Guide for AI Agents

Provisions the resources required to create an S3 backend for Terraform.

## Components

### S3 Bucket
- No public access
- Versioning enabled
- CMK with key rotation and bucket key enabled
- Bucket policy requires secure transport
- Name format "${local.name_prefix}-tfstate-${local.account_id}

### DynamoDB Table
- CMK with key rotation enabled
- Minimal read/write capacity
- Name format "${local.name_prefix}-tfstate-${local.account_id}-lock

---
Source: .ruler/terraform-module.md
---
# Terraform Module Guide for AI Agents

## Core Principles

* **One Responsibility:** Each module should have a single, clear purpose.
* **Composition over Configuration:** Build focused modules that work together, not all-in-one solutions.
* **Minimal, Predictable Interfaces:** Expose only essential variables and outputs. Avoid leaking implementation details.
* **Fail Fast:** Validate inputs and stop early on errors.
* **Clear Documentation:** Explain purpose, usage, and design decisions.

## Module Design

**Good Examples:**

```hcl
# Single-purpose modules
module "vpc" { source = "./modules/vpc" name = var.name cidr_block = var.cidr }
module "db" { source = "./modules/rds" name = "${var.name}-db" subnets = module.vpc.db_subnets }
```

**Avoid:**

```hcl
module "monolith" { create_vpc = true; create_db = true; create_lb = true; ... }
```

## Interfaces

* **Inputs:** Only what's needed; group related config in objects; use sensible defaults and validation.
* **Feature Flags:** Use booleans for optional resources.
* **Outputs:** Only what's useful for composition—IDs, endpoints, etc.
  *Don’t expose internal resource details.*

## Patterns

* **Consistent Naming & Tagging:** Use predictable names and common tags.
* **Version Pinning:** Lock Terraform and provider versions in `versions.tf`.
* **Centralize Data Sources:** Place all external data lookups in one file and pass results to submodules.

## File Structure

```
/main.tf           # Core logic
/variables.tf      # Inputs
/outputs.tf        # Outputs
/locals.tf         # Computed values
/versions.tf       # Version constraints
/dependencies.tf   # Data sources
/examples/         # Usage demos
/modules/          # Optional submodules
/test/             # Automated tests
```

## AWS Best Practices

* Use `aws_iam_policy_document` instead of inline JSON for policies.
* Default to encryption with customer-managed KMS.
* Create log groups explicitly.

## Avoid

* **God Modules:** Don’t build modules that do everything.
* **Too Many Inputs:** Limit options; use environment-based defaults.
* **Implicit Dependencies:** Accept IDs as inputs, not naming assumptions.
* **Tight Coupling:** Modules shouldn’t assume other modules’ structure.

## Documentation & Testing

* Provide clear examples and rationale in `README.md`.
* Use `terraform-docs` for reference docs.
* Ensure modules are easily testable (`terraform plan` with example configs).

## Release & CI/CD

* Tag releases on `main` with semantic versioning.
* Use GitHub Actions (or similar) to run format, validate, and test steps automatically.

## AI Agent Checklist

* [ ] Single responsibility & clear purpose
* [ ] Minimal, validated inputs and meaningful outputs
* [ ] Predictable naming/tagging
* [ ] Centralized external dependencies
* [ ] Example configs and docs provided
* [ ] Tested for `plan` success and edge cases

---

**Summary:**
Favor small, focused modules with clear interfaces and strong defaults. Compose modules for complex scenarios. Document intent and usage. Avoid monolithic, over-configured designs. **Simplicity and clarity beat feature-completeness.**

---
Source: .ruler/terraform-style.md
---
# Terraform Style Guide for AI Agents

*Based on the [Gruntwork](https://docs.gruntwork.io/guides/style/terraform-style-guide/) and [HashiCorp](https://developer.hashicorp.com/terraform/language/style) guides. Focus: simple, reasoning-friendly infrastructure.*

## Core Principles

* **Choose simple over easy:** Each component should have one clear responsibility; prefer clarity over shortcuts.
* **Explicit over implicit:** Make all dependencies and resource relationships obvious.
* **Localize change:** Changes should not require understanding distant code.
* **Composition over configuration:** Build small, independent modules with clear interfaces and minimal assumptions.

## Understanding Terraform Behavior

* Be able to answer:

  * What will be created?
  * In what order?
  * What changes if a variable changes?
  * What external dependencies exist?
* Make dependencies explicit (avoid hidden dependencies through naming/data sources).
* Ensure changes to variables affect only what is expected.

## Design Patterns

* **Module boundaries:** Each module should have a single reason to change (e.g., VPC, compute).
* **Interface design:** Use variables/outputs to define clear, minimal contracts.
* **Data flow:** Ensure values flow in one direction and are derived in locals when needed.

## Syntax and Style

* **Formatting:** Enforced by `terraform fmt` (2-space indent, 120-char lines, blank lines between resources, trailing newline).
* **Naming:** Use `snake_case` and descriptive names with consistent prefixes.
* **Comments:** Use `#` for comments, `----` for section headers, and always explain non-obvious code.
* **Variables:** Always provide descriptions and types; prefer explicit objects over free-form maps; add validation as needed.

## Avoiding Complexity

* **No implicit dependencies:** Reference resources directly, not by naming convention.
* **Avoid nested conditionals:** Use locals to simplify boolean logic.
* **No magic values:** Use variables and data sources, not hardcoded strings or numbers.
* **Single-purpose modules:** Avoid modules overloaded with flags for optional features.

## AI Agent Guidelines

* **Prioritize:** Reasoning clarity, localized change, explicit dependencies, clean interfaces, correct syntax.
* **Generate self-explanatory code:** Comment rationale for settings and explain resource decisions.
* **When refactoring:** Preserve behavior, extract complexity to locals, make dependencies explicit, and clarify logic.
* **Testing:** Verify behavioral correctness, dependency resolution, and predictable change impact.

---

**Remember:** Simple code enables reliable change and long-term maintainability.

---
Source: .ruler/terraform-test.md
---
# Terraform Test Guide for AI Agents

This guide emphasizes creating simple, maintainable tests that validate essential Terraform module behavior using Terratest.

## Testing Philosophy

### What We're Actually Testing
Terraform tests validate the **contract** of your module, not AWS services:
- **Configuration validity**: Does the module accept inputs correctly?
- **Resource planning**: Does Terraform generate the expected plan?
- **Output correctness**: Do outputs match expected values?
- **Edge case handling**: How does the module behave with unusual inputs?

### Simple vs Comprehensive
- **Simple**: Tests focus on one concern and have clear pass/fail criteria
- **Comprehensive**: Tests try to cover every possible scenario and become brittle
- **Choose simple**: Write tests that are easy to understand and maintain

### Plan-Only Testing Philosophy
```go
// Good - Tests the contract without infrastructure costs
planOutput := terraform.Plan(t, terraformOptions)
assert.Contains(t, planOutput, "aws_vpc.main")

// Avoid - Tests AWS service behavior, not module logic
terraform.Apply(t, terraformOptions)
vpcId := terraform.Output(t, terraformOptions, "vpc_id")
// This tests AWS, not your module
```

## Test Design Principles

### Single Responsibility
Each test should validate **one aspect** of module behavior:

```go
// Good - Tests default behavior
func TestVPCDefaults(t *testing.T) {
    // Test that module works with minimal configuration
}

// Good - Tests feature flag
func TestVPCWithNATGateway(t *testing.T) {
    // Test that nat_gateway flag works correctly
}

// Avoid - Tests multiple unrelated features
func TestVPCEverything(t *testing.T) {
    // Tests defaults AND NAT gateway AND custom CIDR AND...
}
```

### Predictable Failure
Test failures should clearly indicate what went wrong:

```go
// Good - Clear assertion with context
assert.Contains(t, planOutput, "aws_nat_gateway.main", 
    "NAT gateway should be created when nat_gateway_enabled=true")

// Avoid - Unclear assertion
assert.NotEmpty(t, planOutput)
```

### Test Independence
Tests should not depend on each other or external state:

```go
// Good - Each test uses unique names
func TestVPCDefaults(t *testing.T) {
    uniqueName := fmt.Sprintf("test-vpc-%d", time.Now().Unix())
    terraformOptions := &terraform.Options{
        Vars: map[string]interface{}{
            "name": uniqueName,
        },
    }
}
```

### Essential vs Incidental Testing
Focus on **essential complexity** (what the module should do) rather than **incidental complexity** (implementation details):

```go
// Essential - Tests the module's contract
assert.Contains(t, planOutput, "cidr_block = \"10.0.0.0/16\"")

// Incidental - Tests implementation details
assert.Contains(t, planOutput, "depends_on = [aws_internet_gateway.main]")
```

## Repository Organization

### Structure Purpose
Each directory serves a single, clear purpose:

```
project/
├── modules/                 # The actual Terraform modules
├── examples/               # Demonstration of module usage
│   ├── defaults/          # Minimal viable configuration
│   └── complete/          # Full feature demonstration
└── test/                  # Validation of examples
    ├── go.mod
    ├── shared_test.go     # Common utilities
    ├── defaults_test.go   # Tests for defaults example
    └── complete_test.go   # Tests for complete example
```

### Example Directory Design
Each example should demonstrate **one usage pattern**:

**defaults/** - Minimal Configuration
```hcl
# Demonstrates basic functionality with sensible defaults
module "vpc" {
  source = "../../modules/vpc"
  
  name = var.name
  # All other values use module defaults
}
```

**complete/** - Full Configuration
```hcl
# Demonstrates all available options
module "vpc" {
  source = "../../modules/vpc"
  
  name                   = var.name
  cidr_block            = var.cidr_block
  enable_nat_gateway    = var.enable_nat_gateway
  availability_zones    = var.availability_zones
  # ... all other options
}
```

### Required Files Per Example
- **main.tf**: Module instantiation and providers
- **variables.tf**: Input definitions with validation
- **outputs.tf**: Output definitions (may be empty)
- **versions.tf**: Version constraints
- **terraform.auto.tfvars**: Values for required variables
- **README.md**: Usage documentation

## Implementation Patterns

### Test Structure Template
```go
func TestModuleExample(t *testing.T) {
    t.Parallel()
    
    // Generate unique test identifier
    testID := fmt.Sprintf("test-%d", time.Now().Unix())
    
    // Configure Terraform options
    terraformOptions := &terraform.Options{
        TerraformDir: "../examples/defaults",
        Vars: map[string]interface{}{
            "name": testID,
        },
        RetryableTerraformErrors: getRetryableErrors(),
        MaxRetries: 3,
        TimeBetweenRetries: 5 * time.Second,
    }
    
    // Ensure cleanup
    defer terraform.Destroy(t, terraformOptions)
    
    // Execute test
    terraform.Init(t, terraformOptions)
    planOutput := terraform.Plan(t, terraformOptions)
    
    // Validate results
    assert.NotEmpty(t, planOutput)
    assert.Contains(t, planOutput, "Changes to Outputs")
}
```

### Shared Utilities (test/shared_test.go)
```go
package test

import (
    "fmt"
    "time"
    "github.com/gruntwork-io/terratest/modules/terraform"
)

// generateTestName creates unique, AWS-compatible names
func generateTestName(prefix string) string {
    timestamp := time.Now().Unix()
    return fmt.Sprintf("%s-%d", prefix, timestamp)
}

// getBaseTerraformOptions provides common configuration
func getBaseTerraformOptions(terraformDir string) *terraform.Options {
    return &terraform.Options{
        TerraformDir:             terraformDir,
        RetryableTerraformErrors: getRetryableErrors(),
        MaxRetries:               3,
        TimeBetweenRetries:       5 * time.Second,
    }
}

// getRetryableErrors defines which errors should trigger retries
func getRetryableErrors() map[string]string {
    return map[string]string{
        // Network issues
        ".*timeout.*":                        "Network timeout",
        ".*connection reset.*":               "Connection reset",
        ".*no such host.*":                   "DNS resolution failure",
        
        // AWS throttling
        ".*Throttling.*":                     "AWS API throttling",
        ".*RequestLimitExceeded.*":           "Request limit exceeded",
        ".*TooManyRequestsException.*":       "Too many requests",
        
        // Temporary resource issues
        ".*ResourceNotReady.*":               "Resource not ready",
        ".*InvalidParameterValue.*subnet.*": "Subnet not yet available",
    }
}
```

### Test Categories

#### Configuration Validation Tests
```go
func TestValidConfiguration(t *testing.T) {
    t.Parallel()
    
    terraformOptions := getBaseTerraformOptions("../examples/defaults")
    terraformOptions.Vars = map[string]interface{}{
        "name": generateTestName("valid"),
    }
    
    defer terraform.Destroy(t, terraformOptions)
    
    terraform.Init(t, terraformOptions)
    planOutput := terraform.Plan(t, terraformOptions)
    
    // Verify plan succeeds and creates expected resources
    assert.Contains(t, planOutput, "Plan: 3 to add, 0 to change, 0 to destroy")
}
```

#### Feature Flag Tests
```go
func TestFeatureDisabled(t *testing.T) {
    t.Parallel()
    
    terraformOptions := getBaseTerraformOptions("../examples/complete")
    terraformOptions.Vars = map[string]interface{}{
        "name":                generateTestName("disabled"),
        "enable_nat_gateway": false,
    }
    
    defer terraform.Destroy(t, terraformOptions)
    
    terraform.Init(t, terraformOptions)
    planOutput := terraform.Plan(t, terraformOptions)
    
    // Verify NAT gateway is not created when disabled
    assert.NotContains(t, planOutput, "aws_nat_gateway")
}
```

#### Input Validation Tests
```go
func TestInvalidInput(t *testing.T) {
    t.Parallel()
    
    terraformOptions := getBaseTerraformOptions("../examples/defaults")
    terraformOptions.Vars = map[string]interface{}{
        "name":       generateTestName("invalid"),
        "cidr_block": "invalid-cidr", // This should cause validation failure
    }
    
    terraform.Init(t, terraformOptions)
    
    // Expect plan to fail with validation error
    _, err := terraform.PlanE(t, terraformOptions)
    assert.Error(t, err)
    assert.Contains(t, err.Error(), "invalid CIDR")
}
```

## Error Handling Strategy

### Retry Logic
Only retry **transient** errors, fail fast on **permanent** errors:

```go
// Transient - Should retry
".*Throttling.*"                    // AWS rate limiting
".*timeout.*"                       // Network issues
".*connection reset.*"              // Temporary connectivity

// Permanent - Should fail immediately  
".*InvalidParameterValue.*"         // Configuration error
".*UnauthorizedOperation.*"         // Permission issue
".*ValidationException.*"           // Input validation error
```

### Error Categories
1. **Network/Connectivity**: Temporary infrastructure issues
2. **AWS Throttling**: Rate limiting that resolves automatically
3. **Resource Dependencies**: Resources not yet available
4. **Configuration Errors**: Should fail immediately without retry

## CI/CD Integration

### Environment Setup
```yaml
# GitHub Actions example
env:
  AWS_DEFAULT_REGION: us-west-2
  TF_VAR_name_prefix: "ci-test"
  TF_VAR_environment: "ci"
```

### Test Execution
```bash
cd test
go mod tidy
go test -v -timeout 30m -parallel 4
```

### Cost Management
- Use plan-only testing to avoid AWS charges
- Set timeouts to prevent runaway tests
- Use unique naming to avoid resource conflicts

## AI Agent Guidelines

### Test Analysis
When examining existing tests, evaluate:

1. **Purpose Clarity**: Can you understand what each test validates?
2. **Failure Scenarios**: Will test failures clearly indicate the problem?
3. **Independence**: Can tests run in any order without conflicts?
4. **Essential Focus**: Do tests validate module contracts, not implementation details?

### Test Generation Principles

#### Start with Module Contract
```go
// Good - Tests what the module promises to do
func TestVPCCreatesSubnets(t *testing.T) {
    t.Parallel()
    
    terraformOptions := getBaseTerraformOptions("../examples/defaults")
    // ... setup
    
    planOutput := terraform.Plan(t, terraformOptions)
    
    // Verify the module creates the promised subnets
    assert.Contains(t, planOutput, "aws_subnet.private")
    assert.Contains(t, planOutput, "aws_subnet.public")
}
```

#### Test Edge Cases Systematically
```go
// Test boundary conditions
func TestVPCWithMinimalCIDR(t *testing.T) {
    // Test with /28 CIDR (smallest practical)
}

func TestVPCWithLargeCIDR(t *testing.T) {
    // Test with /16 CIDR (largest common)
}

func TestVPCWithAllAZs(t *testing.T) {
    // Test with maximum availability zones
}
```

#### Generate Meaningful Test Names
```go
// Good - Name describes the scenario and expected outcome
func TestVPCWithNATGatewayCreatesInternetAccess(t *testing.T) {}
func TestVPCWithoutNATGatewayBlocksPrivateInternet(t *testing.T) {}

// Avoid - Generic names that don't explain the test
func TestVPCNAT(t *testing.T) {}
func TestVPCComplete(t *testing.T) {}
```

### Test Maintenance
When updating tests:

1. **Preserve Intent**: Understand what the original test was validating
2. **Update Assertions**: Ensure assertions still test the right behavior  
3. **Check Dependencies**: Verify test utilities still work correctly
4. **Validate Isolation**: Ensure tests don't interfere with each other

### Debugging Test Failures
Systematic approach to test failures:

1. **Read the Error**: What specific assertion failed?
2. **Check the Plan**: What did Terraform actually try to create?
3. **Verify Inputs**: Are test variables correct?
4. **Isolate the Issue**: Can you reproduce with `terraform plan` manually?
5. **Check Dependencies**: Are required resources/permissions available?

### Common Anti-Patterns to Avoid

#### Over-Testing
```go
// Avoid - Testing every possible input combination
func TestAllPossibleConfigurations(t *testing.T) {
    // This becomes unwieldy and doesn't add value
}

// Better - Test representative scenarios
func TestCommonConfigurations(t *testing.T) {
    // Test the most important use cases
}
```

#### Implementation Testing
```go
// Avoid - Testing Terraform internals
assert.Contains(t, planOutput, "depends_on")

// Better - Testing module behavior
assert.Contains(t, planOutput, "aws_vpc.main")
```

#### Complex Test Logic
```go
// Avoid - Tests that are hard to understand
func TestComplexScenario(t *testing.T) {
    if condition1 && condition2 {
        // Test scenario A
    } else if condition3 {
        // Test scenario B  
    } else {
        // Test scenario C
    }
}

// Better - Separate tests for each scenario
func TestScenarioA(t *testing.T) { /* Clear test logic */ }
func TestScenarioB(t *testing.T) { /* Clear test logic */ }
func TestScenarioC(t *testing.T) { /* Clear test logic */ }
```

## Key Takeaways

1. **Test the Contract**: Focus on what your module promises to do
2. **Keep It Simple**: Each test should have a single, clear purpose
3. **Make Failures Clear**: Test failures should immediately indicate the problem
4. **Plan-Only When Possible**: Avoid unnecessary AWS costs and complexity
5. **Design for Independence**: Tests should not depend on each other
6. **Focus on Essential Behavior**: Test what matters, not implementation details

Remember: The goal is not comprehensive coverage, but confidence in essential functionality.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kbrockhoff) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-14 -->
