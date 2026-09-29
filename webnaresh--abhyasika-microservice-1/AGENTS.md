# GitHub Copilot Instructions for `abhyasika-api`

## Project Overview

The `abhyasika-api` project is a backend application built using the NestJS framework, with Prisma as the ORM. It includes support for AWS S3 integration, JWT-based authentication, and a variety of other features for building scalable APIs.

## Key Features

- **NestJS Framework**: Modular, extensible architecture.
- **Prisma**: Used for database interaction and migrations.
- **Swagger**: API documentation generation.
- **AWS SDK**: For S3 file storage integration.
- **Validation**: Using `class-validator` and `class-transformer`.

## Development Scripts

- `dev`: Starts the NestJS application in watch mode.
- `build`: Builds the application.
- `postbuild`: Runs Prisma's code generation after building.
- `format`: Formats code using Prettier.
- `start`: Starts the application.
- `start:debug`: Starts the application in debug mode.
- `start:prod`: Starts the application in production mode.
- `lint`: Runs ESLint and fixes issues automatically.
- `test`: Runs Jest unit tests.
- `test:watch`: Runs Jest in watch mode.
- `test:cov`: Generates test coverage reports.
- `test:e2e`: Runs end-to-end tests using Jest.
- `prisma:generate`: Generates Prisma client.
- `prisma:migrate`: Applies Prisma migrations.
- `start:build`: Builds and runs the application.
- `postinstall`: Automatically generates Prisma client after dependencies are installed.

## Guidelines for GitHub Copilot

### General Suggestions

- **Follow the NestJS conventions**: Use decorators and dependency injection patterns.
- **Adhere to coding standards**: Format code with Prettier and lint using ESLint.
- **Utilize Prisma**: For database schema management, migrations, and type-safe queries.

### Writing New Features

- **API Endpoints**:

  - Use `@nestjs/common` decorators (`@Controller`, `@Get`, `@Post`, etc.) for defining endpoints.
  - Validate incoming data using `class-validator`.

- **Database Integration**:

  - Use Prisma for all database operations. Ensure models are updated in `schema.prisma` and migrations are applied using `prisma migrate dev`.

- **Service Layer**:

  - Implement core logic in the service layer (`@Injectable()` classes).

- **Testing**:
  - Write unit tests for new features using Jest.
  - Add end-to-end tests for critical flows.

### Best Practices for Dependencies

- Use `@nestjs/swagger` for API documentation.
- Handle file uploads with `@nestjs/platform-express` and `multer`.
- Secure routes with `jsonwebtoken`.
- Avoid over-fetching data by designing optimized Prisma queries.

### Debugging and Troubleshooting

- Use `start:debug` for runtime debugging.
- Inspect logs and errors in Prisma using the `DEBUG` environment variable.

## Recommendations for GitHub Actions

- **Linting**:
  - Run ESLint checks on all PRs.
- **Testing**:
  - Execute Jest unit tests and generate coverage reports.
- **Build and Deploy**:
  - Automate deployment using Prisma migrations and `npm run start:prod`.

## Documentation Requirements

- Ensure new API endpoints are documented in Swagger.
- Update README.md with details of any new environment variables or setup steps.

---

This file is a reference for developers and contributors to guide their use of GitHub Copilot while working on `abhyasika-api`. For additional help, consult the project's README or reach out to maintainers.

---
> Source: [WebNaresh/abhyasika_microservice_1](https://github.com/WebNaresh/abhyasika_microservice_1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-29 -->
