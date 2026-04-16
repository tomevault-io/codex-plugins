# Copilot Instructions for Ve's Portfolio Site

## Overview

This repository contains the source code for Ve's portfolio site, built using modern web technologies like Next.js, React, TypeScript, TailwindCSS, and more. The site is designed to showcase Ve's technical career and includes features like a blog, responsive design, and optimized deployment workflows.

## Key Components

- **Frontend**: Built with Next.js 14 and React, styled using TailwindCSS and Shadcn/UI.
- **Backend**: Minimal backend logic, primarily focused on serving static and dynamic content.
- **Configuration**: Centralized in `src/data/resume.tsx` for easy customization.
- **Testing**: Unit, integration, and E2E tests using Jest and Playwright.
- **Deployment**: Dockerized application with support for Google Cloud Platform (GCP) Artifact Registry and Terraform for infrastructure management.

## Developer Workflows

### Local Development

1. Clone the repository:
    ```bash
    git clone https://github.com/vesharma-dev/ve-portfolio-site.git
    cd ve-portfolio-site
    ```
2. Install dependencies:
    ```bash
    pnpm install
    ```
3. Start the development server:
    ```bash
    pnpm dev
    ```
4. Edit the configuration file at `src/data/resume.tsx` to customize the site.

### Docker Workflow

- Build the Docker image:
    ```bash
    docker build --no-cache -t ve-app .
    ```
- Run the Docker container:
    ```bash
    docker run ve-app
    ```

### Deployment to GCP Artifact Registry

1. Authenticate with GCP:
    ```bash
    gcloud auth login
    ```
2. Tag and push the Docker image:
    ```bash
    docker tag ve-app gcr.io/experimental-learning/ve-app:latest
    docker push gcr.io/experimental-learning/ve-app:latest
    ```

### Infrastructure Management with Terraform

- Initialize Terraform:
    ```bash
    terraform init
    ```
- Plan and apply changes:
    ```bash
    terraform plan
    terraform apply
    ```
- Destroy resources:
    ```bash
    terraform destroy
    ```

### Testing

#### Unit & Integration Tests

- Run tests and generate a coverage report:
    ```bash
    pnpm run test
    ```
- Validate CircleCI configuration:
    ```bash
    pnpm validate-circle-ci
    ```

#### E2E Tests

- Run E2E tests:
    ```bash
    pnpm run e2e
    ```

## Project-Specific Conventions

- **Configuration**: All major site settings are centralized in `src/data/resume.tsx`.
- **Styling**: TailwindCSS is used for styling, with utility-first class names.
- **Testing**: Jest is configured for both frontend and backend tests, with TypeScript support.
- **E2E Framework**: Playwright is used for end-to-end testing.

## External Dependencies

- **Google Cloud Platform**: Used for hosting Docker images in Artifact Registry.
- **Terraform**: Manages infrastructure as code.
- **CircleCI**: Handles CI/CD pipelines.
- **Shortest Framework**: Provides AI-driven E2E testing capabilities.
- **Shortest Framework**: Provides AI-driven E2E testing capabilities.

## Notes for AI Agents

- Focus on maintaining the centralized configuration in `src/data/resume.tsx`.
- Follow the established testing workflows for Jest and Playwright.
- Ensure Docker and Terraform commands are used as described for deployment and infrastructure management.
- Adhere to TailwindCSS conventions for styling.

## TODOs

- Add Web3 functionality to enable minting of Ve Coins.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VeVarunSharma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
