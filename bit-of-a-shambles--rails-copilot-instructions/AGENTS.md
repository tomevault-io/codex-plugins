# Modern Rails 8 Application

This is a Rails 8 application built with modern practices including Turbo, Stimulus, and Solid Queue. Follow these guidelines for all code generation:

## General Coding Guidelines

- Use semantic and clean HTML in ERB templates
- Follow Ruby style conventions (2-space indentation, snake_case methods)
- Use Turbo for page transitions and dynamic updates
- Implement Stimulus controllers for interactive elements
- Utilize Tailwind CSS for styling with component-based design
- Keep controllers thin, move logic to service objects when appropriate
- Use Import Maps for JavaScript dependency management
- Use Solid Queue for background job processing
- Follow Kamal deployment best practices for production

## Technology Stack

- Rails 8.0
- Ruby 3.3+
- Turbo & Stimulus (Hotwire)
- Solid Queue
- Kamal for deployment
- SQLite
- Tailwind CSS
- Import Maps for JavaScript management

## Project Architecture

This application follows a standard Rails structure with some specific organization:

- Controllers are minimal and focused on presentation
- Service objects handle complex business logic
- Background jobs process asynchronous tasks
- Use Turbo and Stimulus for interactive components
- Prefer Import Maps over bundlers for JavaScript dependency management
- Follow Rails conventions for file structure and naming
- Use SQLite for development and PostgreSQL for production

## Specialized Instruction Files

This application uses specialized instruction files to define best practices for specific areas:

- Rails 8: See `.github/instructions/rails8.instructions.md` for Rails 8 conventions
- Turbo/Stimulus: See `.github/instructions/turbo-stimulus.instructions.md` for Hotwire patterns
- Solid Queue: See `.github/instructions/solid-queue.instructions.md` for background job processing
- Kamal: See `.github/instructions/kamal.instructions.md` for deployment configuration
- TailwindCSS: See `.github/instructions/tailwind.instructions.md` for styling guidelines
- Import Maps: See `.github/instructions/importmaps.instructions.md` for JS dependency management

---
> Source: [bit-of-a-shambles/rails_copilot_instructions](https://github.com/bit-of-a-shambles/rails_copilot_instructions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
