# Task Management System - Copilot Instructions

## Architecture Overview

This is a **cloud-native, microservices-based task management system** with full DevOps automation:

- **task-api/**: REST API backend service (Python/Flask)
- **task-cli/**: Command-line interface client
- **jenkins/**: Multi-pipeline CI/CD orchestration
- **terraform/**: AWS infrastructure provisioning (Jenkins, K8s cluster)
- **ansible/**: Configuration management automation
- **k8s/**: Kubernetes deployment manifests

### CI/CD Flow
```
GitHub Push → Jenkins Master → Parallel CI (API/CLI) → CD Pipeline → K8s Deployment
```

## Project Structure

```
task-api/                    # Backend REST API service
├── app.py                   # Flask application with /api/v1/* endpoints
├── models.py                # Database models (Task schema)
├── version.py               # Semantic version tracking
├── Dockerfile               # Multi-stage Docker build
└── tests/                   # pytest test suite

task-cli/                    # CLI client tool
├── task_cli/
│   ├── cli.py               # Click commands (list, add, update, delete)
│   ├── api_client.py        # HTTP client for /api/v1/* endpoints
│   └── version.py           # Must stay compatible with API version
├── setup.py                 # Installable package (console_scripts)
└── tests/                   # Mock-based CLI tests

jenkins/                     # Pipeline definitions
├── Jenkinsfile.master       # Orchestrates version detection & triggers
├── Jenkinsfile.api          # API CI: test → build → push → tag
├── Jenkinsfile.cli          # CLI CI: test → package → version check
└── Jenkinsfile.deploy       # CD: compatibility check → K8s deploy

terraform/                   # Infrastructure as Code
├── jenkins/                 # Jenkins server provisioning
│   ├── main.tf              # EC2, security groups, IAM roles
│   ├── user-data.sh         # Bootstrap script for Jenkins
│   └── variables.tf
└── k8s-cluster/             # EKS or self-managed K8s cluster
    └── main.tf

ansible/                     # Configuration management
├── playbooks/
│   ├── jenkins-setup.yml    # Install plugins, configure jobs
│   └── k8s-setup.yml        # Cluster initialization
└── roles/                   # Reusable automation roles

k8s/                         # Kubernetes manifests
├── namespace.yaml           # task-management namespace
├── deployment.yaml          # task-api deployment (3 replicas)
├── service.yaml             # LoadBalancer service
└── ingress.yaml             # External access configuration
```

## Key Architectural Decisions

### Service Separation & Communication
- **API** handles all business logic and data persistence (Flask + SQLAlchemy)
- **CLI** is a thin client calling REST endpoints (no direct DB access)
- Communication: `CLI → HTTP/REST → API → Database`
- API base URL: `TASK_API_URL` env var (default: `http://localhost:5000`, prod: K8s LoadBalancer)

### API Endpoints (RESTful Design)
```
GET    /api/v1/tasks          # List all tasks
POST   /api/v1/tasks          # Create task (JSON body)
GET    /api/v1/tasks/:id      # Get specific task
PUT    /api/v1/tasks/:id      # Update task
DELETE /api/v1/tasks/:id      # Delete task
GET    /api/v1/health         # Health check (for K8s probes)
GET    /api/v1/version        # Returns API version from version.py
```

### Version Management Strategy
- **Semantic Versioning**: MAJOR.MINOR.PATCH
- API and CLI versions tracked independently in `version.py` files
- Jenkins master pipeline detects version changes by comparing git commits
- CLI must remain compatible with API (check in `Jenkinsfile.deploy`)
- Version tags applied after successful CI: `api-v1.2.3`, `cli-v1.2.0`

### CI/CD Pipeline Architecture
1. **Jenkins Master** (`Jenkinsfile.master`): Detects changes (API/CLI/both), triggers parallel builds
2. **API CI** (`Jenkinsfile.api`): pytest → Docker build → push to registry → git tag
3. **CLI CI** (`Jenkinsfile.cli`): pytest → package build → version validation → git tag
4. **Deploy CD** (`Jenkinsfile.deploy`): Version compatibility check → kubectl apply → health checks

## Development Workflows

### Local Development Setup
```bash
# API (runs on port 5000)
cd task-api
python -m pip install -r requirements.txt
python app.py

# CLI (install in editable mode)
cd task-cli
pip install -e .
task-cli --help
export TASK_API_URL=http://localhost:5000  # Point to local API
```

### Testing
```bash
# API tests (use Flask test client, no external dependencies)
cd task-api && python -m pytest tests/ -v

# CLI tests (mock api_client.py responses, no actual API calls)
cd task-cli && python -m pytest tests/ -v
```

### Docker Local Testing
```bash
# Build and run API container
cd task-api
docker build -t task-api:dev .
docker run -p 5000:5000 task-api:dev

# Build CLI container (if needed for testing)
cd task-cli
docker build -t task-cli:dev .
```

### Infrastructure Provisioning
```bash
# Terraform: Provision Jenkins server
cd terraform/jenkins
terraform init
terraform plan
terraform apply

# Terraform: Provision K8s cluster on AWS
cd terraform/k8s-cluster
terraform apply

# Ansible: Configure Jenkins (plugins, jobs, credentials)
cd ansible
ansible-playbook playbooks/jenkins-setup.yml -i inventory/
```

### Kubernetes Deployment
```bash
# Apply all manifests (namespace, deployment, service, ingress)
kubectl apply -f k8s/

# Check deployment status
kubectl get pods -n task-management
kubectl logs -n task-management deployment/task-api

# Port-forward for local testing
kubectl port-forward -n task-management svc/task-api 5000:5000
```

## Code Conventions

### API (task-api/)
- Use **Flask** or **FastAPI** for REST endpoints
- `models.py`: Define SQLAlchemy models or Pydantic schemas
- RESTful URL patterns: `/tasks`, `/tasks/<id>`, etc.
- Return JSON responses with appropriate HTTP status codes
- Include `version.py` with `__version__` variable for API versioning

### CLI (task-cli/)
- Use **Click** framework for command definitions (preferred) or argparse
- Commands should mirror API operations: `task-cli list`, `task-cli add`, `task-cli delete <id>`
- `api_client.py` should handle all HTTP communication with error handling
- CLI should provide user-friendly error messages when API is unreachable
- Package as installable tool via `setup.py` with console_scripts entry point

### Testing Patterns
- **API tests** (`test_api.py`): Use Flask test client, mock database if needed
- **Model tests** (`test_models.py`): Test SQLAlchemy model validation and relationships
- **CLI tests** (`test_cli.py`): Mock `api_client.py` responses using `@patch`, no actual HTTP calls
- All tests must pass in CI before Docker build/package creation

### Version Management
- **Format**: `__version__ = "MAJOR.MINOR.PATCH"` in both `version.py` files
- **Bumping**: Manual version increment triggers CI/CD (detected by Jenkins master)
- **Tagging**: Automatic git tags created post-CI: `api-v1.2.3`, `cli-v1.2.0`
- **Compatibility**: CLI must support current API version (checked in deploy pipeline)

## Dependencies

### Expected in task-api/requirements.txt
```
flask or fastapi
sqlalchemy (if using database)
pytest
```

### Expected in task-cli/requirements.txt
```
click
requests
pytest
```

## Common Development Tasks

### Adding a New API Endpoint
1. Define route in `task-api/app.py` (e.g., `@app.route('/api/v1/tasks/<id>', methods=['PUT'])`)
2. Add/update model in `task-api/models.py` if schema changes
3. Write test in `task-api/tests/test_api.py` using Flask test client
4. Update API version in `task-api/version.py` if breaking change

### Adding a New CLI Command
1. Add Click command in `task-cli/task_cli/cli.py` (e.g., `@click.command()`)
2. Implement HTTP call in `task-cli/task_cli/api_client.py` (call new endpoint)
3. Write test in `task-cli/tests/test_cli.py` mocking `api_client` responses
4. Update CLI version in `task-cli/task_cli/version.py` if needed

### Modifying Infrastructure
- **Jenkins changes**: Update relevant `Jenkinsfile.*` in `jenkins/`
- **K8s changes**: Modify manifests in `k8s/`, apply with `kubectl apply -f k8s/`
- **AWS changes**: Update Terraform in `terraform/`, run `terraform plan && terraform apply`
- **Server config**: Update Ansible playbooks in `ansible/playbooks/`

## CI/CD Integration Points

### Jenkins Pipeline Triggers
- **Webhook**: GitHub push to `main` triggers `Jenkinsfile.master`
- **Version Detection**: Master compares `version.py` files between commits
- **Parallel Execution**: API and CLI pipelines run simultaneously if both changed
- **Deployment Gate**: CD pipeline only runs after successful CI for both components

### Docker Registry Flow
- **API**: `task-api:latest` and `task-api:v1.2.3` pushed to registry after CI
- **CLI**: Packaged as wheel/egg, optionally containerized
- **K8s Pull**: Deployment pulls `task-api:latest` from registry

### Kubernetes Health Checks
- **Liveness probe**: `GET /api/v1/health` (restart pod if fails)
- **Readiness probe**: `GET /api/v1/health` (remove from load balancer if fails)
- **Deployment strategy**: Rolling update (maxSurge: 1, maxUnavailable: 0)

## License
Apache License 2.0 - See LICENSE file

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RazielRey)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/RazielRey)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
