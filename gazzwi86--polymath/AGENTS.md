<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

<!--
  Copilot Custom Instructions for this Workspace
  Project: Polymath (Tech Radar Invite Agent)
-->

This project is a production-grade TypeScript LangChain agent for generating Tech Radar meeting invite blurbs, designed for cloud-native, serverless deployment.

**Tech Stack & Tooling:**

- TypeScript, Node.js, LangChain (Ollama, Gemini, Bedrock support)
- Observability: LangSmith
- Testing: Jest
- Linting/Formatting: ESLint, Prettier, EditorConfig
- Infrastructure: Terraform (AWS Fargate, ECS, ECR, VPC, API Gateway, IAM, CloudWatch)
- CI/CD: GitHub Actions (build, test, lint, deploy)
- Pre-commit: Husky + lint-staged (format/lint only staged files)
- Dockerized for deployment

**Best Practices:**

- Always use environment variables for API keys, endpoints, and secrets.
- All infrastructure IDs (VPC, subnets, security groups, roles) are provisioned and referenced in Terraform—do not require manual input.
- Only expose and require the minimal set of variables for configuration.
- Prefer stateless, ephemeral, serverless patterns (Fargate, API Gateway, etc).
- HTTP server should listen on the port defined by the `container_port` variable (default 3000).
- All code should be clean, idiomatic TypeScript and pass lint, format, and test checks before commit/push.

**Copilot, when generating code:**

- Use modern TypeScript and Node.js idioms.
- Use async/await for all async code.
- Use environment variables for all secrets and endpoints.
- Prefer functional, stateless patterns unless state is required.
- For infra, always reference Terraform resource outputs instead of requiring IDs as variables.
- For new features, add tests and update documentation/readme as needed.
- For HTTP endpoints, use Express or Fastify and listen on the configured port.
- For CI/CD, ensure all steps (build, lint, test, deploy) are covered.
- For Docker, EXPOSE the correct port and use a minimal, secure base image.

**Do not:**

- Hardcode secrets, API keys, or endpoints.
- Require manual wiring of AWS resource IDs that can be provisioned in Terraform.
- Add files or dependencies not relevant to the stack above.

**Reference:**

- See README.md for architecture, deployment, and usage details.
- See infra/ for all Terraform infrastructure code.
- See src/ for all agent logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gazzwi86) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
