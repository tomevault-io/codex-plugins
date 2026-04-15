# GitHub Copilot Instructions

This file provides context and guidelines for GitHub Copilot when working on the Tally project.

## Project Overview

Tally is a financial application for managing recurring bills and forecasting bank account balances. The architecture consists of:

- **Backend**: Python FastAPI application deployed on AWS Lambda
- **Frontend**: Svelte application (planned) served via AWS CloudFront and S3
- **Database**: PostgreSQL on AWS RDS
- **Authentication**: Auth0 integration
- **Infrastructure**: Terraform for AWS resource management

## Cost Management Philosophy

**This project prioritizes cost optimization for solo developers without VC funding.**

### Cost-First Design Principles

- **Zero unnecessary AWS costs**: Eliminate NAT Gateways, unused Elastic IPs, and expensive services
- **Leverage AWS Free Tier**: Maximize usage of free services and resource limits
- **Public subnet architecture**: Lambda functions in public subnets (with security groups) vs private + NAT Gateway
- **Conservative resource sizing**: Use minimal CIDR ranges, single-AZ deployments where appropriate
- **Cost-aware alternatives**: Choose managed services only when they provide clear value over self-managed options

### Infrastructure Cost Guidelines

- **VPC Architecture**: Public subnets + security groups instead of private subnets + NAT Gateway (saves $588/year)
- **Database**: Consider RDS Free Tier limits, use db.t3.micro or similar cost-effective instances
- **Lambda**: Stay within generous free tier limits (1M requests, 400,000 GB-seconds per month)
- **Storage**: Use S3 Standard-IA or Glacier for infrequent access, lifecycle policies for cost optimization
- **Monitoring**: Use CloudWatch Free Tier, avoid unnecessary custom metrics and dashboards

## Code Style and Conventions

### Python (Backend)

- Use Python 3.13+ features
- Follow PEP 8 style guidelines
- Use type hints for all function signatures
- Prefer Poetry for dependency management
- Use pytest for testing
- Follow FastAPI best practices for API development
- Use Pydantic models for request/response validation

### Infrastructure (Terraform)

- Use Terraform >= 1.0.0
- Organize code into reusable modules
- Follow HashiCorp Configuration Language (HCL) best practices
- Use variables.tf and outputs.tf for module interfaces
- Include comprehensive resource tagging
- Use data sources for existing resources when possible
- **Cost optimization first**: Always consider monthly costs before adding resources
- **Favor free-tier resources**: VPC subnets, security groups, route tables are free
- **Avoid cost generators**: NAT Gateways ($45/month), Elastic IPs ($3.65/month), excessive data transfer

### File Formatting

- **Always end files with a single empty line** - This is a POSIX standard and helps with clean diffs
- Use consistent indentation (2 spaces for YAML/HCL, 4 spaces for Python)
- Remove trailing whitespace
- Use Unix line endings (LF)
- Keep line lengths reasonable (80-120 characters)

### Git Workflow

- Use conventional commits format (feat:, fix:, docs:, etc.)
- Create feature branches from main
- Use descriptive branch names (feature/description, fix/bug-name)
- Include tests for new features
- Update documentation when adding new functionality
- **Prefer rebase over merge commits**: Use `git rebase origin/main` instead of `git merge` to maintain clean history
- **Avoid merge commits**: When resolving conflicts, always use rebase (`git rebase origin/main`) instead of merge commits

#### Pull Request Creation

- **For complex PR descriptions**: Create `pr-body.md` file and use `gh pr create --body-file pr-body.md`
- **For simple PRs**: Use inline `--body` with GitHub CLI
- **Always clean up**: Remove `pr-body.md` after PR creation (it's temporary)
- **Use emojis and formatting**: Make PR descriptions clear and scannable with sections, checkboxes, and context

### GitHub Actions Version Management

- All GitHub Actions versions are centrally managed in `.github/action-versions.conf`
- Use `make validate-actions` to check that all workflows use approved versions
- Install git hooks with `make install-git-hooks` to automatically validate on commits
- Update `.github/action-versions.conf` when approving new action versions
- The validation script at `scripts/validate-actions.sh` enforces version consistency

#### Branch Management & Cleanup

- **Auto-prune setup**: Configure `git config --global fetch.prune true` for automatic cleanup
- **Regular pruning**: Use `git remote prune origin` to remove stale remote references
- **Local cleanup**: Delete merged branches with `git branch -d branch-name`
- **Force delete unmerged**: Use `git branch -D branch-name` for branches with open PRs
- **Verify cleanup**: Use `git branch -a` to check remaining branches

## Development Practices

### Local Development

- Use Docker Compose for local development environment
- **Use `.secrets` file** for local sensitive configuration (never commit this file)
- Test GitHub Actions locally using `act` (use Makefile targets like `make github_workflow_terraform-pr`)
- Validate Terraform changes locally before committing
- Run tests before pushing changes
- **Always check costs**: Use `terraform plan` and AWS cost calculators before deploying resources

#### .secrets File Pattern

Create a `.secrets` file in the project root for local development:

```bash
# .secrets file (never commit - already in .gitignore)
AWS_PROFILE=AdministratorAccess-123456789012
AWS_ROLE_ARN=arn:aws:iam::123456789012:role/tally-github-actions-role
TF_VAR_aws_account_id=123456789012
TF_VAR_aws_profile=AdministratorAccess-123456789012
```

Source it in your shell:

```bash
source .secrets
make github_workflow_terraform-pr  # Now has access to AWS credentials
```

### Testing

- Write unit tests for all business logic
- Use pytest fixtures for test setup
- Mock external dependencies in tests
- Aim for high test coverage
- Test both success and error scenarios

### Security

**🔒 CRITICAL: Never commit sensitive information to the repository.**

#### Sensitive Data Protection

- **Never commit secrets, API keys, tokens, or credentials** to any branch
- **Never commit real AWS account IDs, ARNs, or resource identifiers**
- **Never commit personal paths** (e.g., `/Users/username/`) - use generic placeholders
- **Use `.secrets` file** for local development configuration (already in .gitignore)
- **Use GitHub repository secrets** for CI/CD credentials
- **Use placeholder values** in documentation and examples (e.g., `123456789012` for AWS account IDs)

#### Configuration Management

- **Local Development**: Use `.secrets` file for sensitive configuration
- **GitHub Actions**: Use repository secrets (`${{ secrets.SECRET_NAME }}`)
- **Terraform**: Use variables and data sources, never hardcode sensitive values
- **Documentation**: Always use placeholder values, never real credentials

#### Code Examples - DO NOT DO:

```hcl
# ❌ NEVER DO THIS
bucket = "terraform-state-993450011441"  # Real account ID
profile = "AdministratorAccess-993450011441"  # Real account ID
```

#### Code Examples - CORRECT:

```hcl
# ✅ CORRECT - Use variables
bucket = "terraform-state-${var.aws_account_id}"
profile = var.aws_profile

# ✅ CORRECT - Placeholder in docs
bucket = "terraform-state-123456789012"  # Your AWS account ID
```

#### Emergency Response

If sensitive data is accidentally committed:

1. **DO NOT** push the commit
2. Use `git filter-repo` to clean history if already pushed
3. Contact GitHub Support if data is in PR history
4. Rotate any exposed credentials immediately

#### Additional Security Practices

- Follow AWS security best practices
- Use least privilege principle for IAM roles
- Validate all user inputs
- Enable comprehensive logging for security events
- Use environment variables for all configuration

## Project Structure

```
├── backend/          # Python FastAPI application
│   ├── src/         # Source code
│   ├── tests/       # Test files
│   ├── pyproject.toml # Poetry configuration
│   └── Dockerfile   # Container configuration
├── infra/           # Terraform infrastructure
│   ├── modules/     # Reusable Terraform modules
│   ├── main.tf      # Main infrastructure configuration
│   └── Makefile     # Infrastructure automation
├── scripts/         # Utility scripts
├── docs/           # Documentation
└── .github/        # GitHub configuration and workflows
```

## Cost Management Patterns

### Zero-Cost VPC Architecture

```hcl
# ✅ COST-FREE: Public subnets + security groups
resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.main.id
  map_public_ip_on_launch = true
  # Lambda functions run here with security group protection
}

resource "aws_security_group" "lambda" {
  # Provides same security as private subnet + NAT Gateway
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# ❌ EXPENSIVE: Avoid these cost generators
# resource "aws_nat_gateway" "main" {
#   # Costs $45/month + data processing fees
# }
# resource "aws_eip" "nat" {
#   # Costs $3.65/month per IP
# }
```

### Cost-Aware Resource Sizing

```hcl
# ✅ CONSERVATIVE: Minimal CIDR for solo developer
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/20"  # 4,096 IPs vs 65,536
}

# ✅ FREE TIER: Use smallest viable instances
resource "aws_db_instance" "main" {
  instance_class = "db.t3.micro"  # Free tier eligible
  allocated_storage = 20          # Free tier limit
}

# ❌ EXPENSIVE: Avoid over-provisioning
# cidr_block = "10.0.0.0/16"     # 65K IPs - excessive for solo dev
# instance_class = "db.r5.large" # $200+/month
```

### Cost Monitoring Tags

```hcl
# Always include cost tracking tags
resource "aws_lambda_function" "api" {
  tags = {
    Environment = var.environment
    Project     = "tally"
    CostCenter  = "solo-developer"
    Purpose     = "api-backend"
  }
}
```

## Common Patterns

### FastAPI Route Structure

```python
from fastapi import APIRouter, Depends, HTTPException
from pydantic import BaseModel

router = APIRouter(prefix="/api/v1", tags=["resource"])

class ResourceRequest(BaseModel):
    name: str
    value: int

class ResourceResponse(BaseModel):
    id: int
    name: str
    value: int

@router.post("/resources", response_model=ResourceResponse)
async def create_resource(request: ResourceRequest):
    # Implementation here
    pass
```

### Terraform Module Structure

```hcl
# variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
}

# main.tf
resource "aws_lambda_function" "api" {
  function_name = "${var.environment}-tally-api"
  # ... other configuration

  tags = {
    Environment = var.environment
    Project     = "tally"
  }
}

# outputs.tf
output "function_arn" {
  description = "Lambda function ARN"
  value       = aws_lambda_function.api.arn
}
```

### Error Handling

```python
from fastapi import HTTPException

# Use appropriate HTTP status codes
raise HTTPException(
    status_code=404,
    detail="Resource not found"
)

# Log errors appropriately
import logging
logger = logging.getLogger(__name__)
logger.error("Error processing request", exc_info=True)
```

## Environment-Specific Guidelines

### Development

- Use local PostgreSQL via Docker Compose
- Enable debug logging
- Use development-specific configuration
- Mock external services when possible

### Production

- Use AWS RDS for PostgreSQL
- Enable comprehensive logging and monitoring
- Use production-grade security configurations
- Implement proper error handling and recovery

## Dependencies and Tools

### Backend Dependencies

- FastAPI: Web framework
- Pydantic: Data validation
- SQLAlchemy: ORM (if needed)
- pytest: Testing framework
- Poetry: Dependency management

### Infrastructure Tools

- Terraform: Infrastructure as Code
- AWS CLI: AWS operations
- act: Local GitHub Actions testing
- Docker: Containerization

## Documentation Requirements

- Update README.md for user-facing changes
- Update docs/DEVELOPING.md for development process changes
- Include docstrings for all public functions and classes
- Document API endpoints with FastAPI automatic documentation
- Include examples in documentation

## GitHub Actions Workflows

### Workflow Design Principles

**Keep workflows terse and maintainable**:

- **Combine related steps** into single multi-command blocks using `run: |`
- **Use concise job/step names** - prefer `apply` over `Apply Infrastructure Changes`
- **Leverage bash shortcuts** - use `||` and `&&` for conditional logic instead of verbose if-statements
- **Remove unnecessary echo statements** - only include essential user feedback
- **Use inline conditionals** where possible instead of separate conditional steps

#### Terse Workflow Patterns

```yaml
# ✅ GOOD - Terse and clear
- name: Init & Plan
  run: |
    terraform init -input=false
    terraform validate
    terraform plan -out=tfplan
    echo "has_changes=$([[ $? -eq 2 ]] && echo true || echo false)" >> $GITHUB_OUTPUT

# ❌ AVOID - Verbose with unnecessary separation
- name: Terraform Initialize
  run: |
    echo "🔧 Initializing Terraform..."
    terraform init -input=false
    if [ $? -ne 0 ]; then
      echo "❌ Terraform init failed!"
      exit 1
    fi
    echo "✅ Terraform initialized successfully"

- name: Terraform Validate Configuration
  run: |
    echo "🔍 Validating Terraform configuration..."
    terraform validate
    # ... more verbose error handling
```

#### Action Usage Best Practices

- **Use action shortcuts**: `uses: actions/checkout@v5` instead of full step definitions
- **Combine permissions**: Define minimal required permissions at job level
- **Leverage defaults**: Use `defaults.run.working-directory` to avoid repetition
- **Cache strategically**: Only cache when there's measurable benefit

#### Workflow Optimization Techniques

**Multi-command Steps**:

```yaml
# ✅ Combine validation steps
- name: Validate
  run: |
    [ -n "$TF_VAR_environment" ] || { echo "Environment required"; exit 1; }
    aws sts get-caller-identity >/dev/null
    terraform fmt -check -recursive

# ❌ Separate steps add overhead
- name: Check Environment
- name: Validate AWS
- name: Check Formatting
```

**Bash Conditionals**:

```yaml
# ✅ Inline conditional logic
- name: Apply
  run: |
    terraform apply -auto-approve tfplan
    echo "Status: $([[ $? -eq 0 ]] && echo success || echo failed)"

# ❌ Verbose if-statement blocks
- name: Apply
  run: |
    terraform apply -auto-approve tfplan
    if [ $? -eq 0 ]; then
      echo "Success"
    else
      echo "Failed"
    fi
```

**Conditional Step Execution**:

```yaml
# ✅ Use step conditions for major logic branches
- name: Apply Changes
  if: steps.plan.outputs.has_changes == 'true'

- name: Skip (No Changes)
  if: steps.plan.outputs.has_changes == 'false'
```

**Essential Output Only**:

- Remove decorative emojis and verbose progress messages
- Focus on actionable information and error details
- Use structured output for debugging (exit codes, timestamps)
- Prefer `2>/dev/null` to suppress unnecessary warnings

### Current Workflows

#### ci.yml

- Runs on push/PR to main
- Tests backend with Python 3.13 and Poetry
- **~20 lines** - focused on essential testing
- **CRITICAL**: Job name `backend-test` is a required status check for PRs
- **Do not rename** the `backend-test` job without updating branch protection rules

#### terraform-pr.yml

- Validates Terraform on PRs affecting `infra/`
- Posts plan results as PR comments
- Supports both GitHub Actions and local `act` testing
- **~110 lines** - comprehensive validation with PR feedback

#### terraform-apply.yml

- Applies infrastructure changes on main branch pushes
- Uses OIDC authentication with AWS
- Includes concurrency controls and conditional execution
- **~60 lines** - production-ready deployment

Use `make github_workflow_terraform-pr` to test workflows locally before pushing.

### Required Status Checks

**Critical job names that must not be changed:**

- **`backend-test`** (from ci.yml) - Required for all PRs to merge
- Changing this job name will cause PRs to hang waiting for the old job name
- Future jobs should follow the pattern: `frontend-test`, `integration-test`, etc.

**Before renaming any workflow job:**

1. Check if it's configured as a required status check
2. Update branch protection rules if necessary
3. Consider the impact on open PRs (they may need to be rebased)

## GitHub CLI and PR Management

### Accessing PR Comments and Reviews

**CRITICAL: Never use interactive GitHub CLI commands in automation or when programmatic access is needed.**

Use curl with GitHub API for reliable, non-interactive access:

```bash
# ✅ CORRECT - Use curl for PR comments
curl -s -H "Authorization: token $(gh auth token)" \
  "https://api.github.com/repos/kenhowardpdx/tally/pulls/<pr-number>/comments" | \
  jq -r '.[] | "\(.path):\(.line) - \(.body)"'

# ✅ CORRECT - Use curl for PR reviews
curl -s -H "Authorization: token $(gh auth token)" \
  "https://api.github.com/repos/kenhowardpdx/tally/pulls/<pr-number>/reviews"

# ❌ AVOID - Interactive commands that hang in automation
gh pr view <pr-number> --comments  # Opens interactive pager
gh pr view <pr-number>              # Opens interactive interface

# ✅ CORRECT - Non-interactive GitHub CLI usage
gh pr list --head <branch-name>
gh pr status
gh api repos/kenhowardpdx/tally/pulls/<pr-number>/comments
```

### Copilot Review Comments

When Copilot provides review comments on PRs:

1. **Get specific line-by-line feedback** using the API command above
2. **Address each comment systematically** with focused commits
3. **Use conventional commit format** for review fixes (e.g., `fix: address copilot PR review feedback`)

**Example workflow:**

```bash
# Get PR comments for review
curl -s -H "Authorization: token $(gh auth token)" \
  "https://api.github.com/repos/kenhowardpdx/tally/pulls/<pr-number>/comments" | \
  jq -r '.[] | "\(.path):\(.line) - \(.body)"'

# Address significant issues and commit fixes
git add .
git commit -m "fix: address copilot PR review feedback"
git push
```

**Common issues Copilot flags:**

- Redundant code patterns
- Formatting inconsistencies
- Accidental test/debug code
- Security concerns
- Performance optimizations
- Shell compatibility issues
- Missing error handling

---

When suggesting code or infrastructure changes, please follow these guidelines and patterns to maintain consistency across the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kenhowardpdx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
