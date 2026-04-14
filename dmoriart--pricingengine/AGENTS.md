<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# PricingEngine Project Instructions

This is a PySpark-based pricing engine with FastAPI REST API endpoints. The project follows these key principles:

## Architecture
- **Core**: Spark session management and shared utilities
- **Pricing**: Business logic for price calculations using PySpark DataFrames
- **API**: FastAPI-based REST endpoints for pricing operations
- **Config**: Environment-based configuration management
- **Tests**: Comprehensive test coverage for all components

## Key Technologies
- **PySpark**: For distributed data processing and pricing calculations
- **FastAPI**: For high-performance API endpoints with automatic documentation
- **Pydantic**: For data validation and settings management
- **Uvicorn**: ASGI server for running the API

## Code Guidelines
- Use type hints throughout the codebase
- Follow PEP 8 style guidelines
- Implement proper error handling and logging
- Write comprehensive tests for all business logic
- Use dependency injection for testability

## PySpark Best Practices
- Lazy evaluation - defer computations until action is called
- Use DataFrame API for better optimization
- Partition data appropriately for performance
- Cache frequently used DataFrames
- Always stop Spark sessions properly

## API Design
- Use Pydantic models for request/response validation
- Implement proper HTTP status codes
- Include comprehensive error handling
- Provide clear API documentation
- Use async/await for I/O operations

## Environment Configuration
- Use environment variables for configuration
- Support multiple environments (dev, staging, prod)
- Keep sensitive data in environment files
- Validate configuration on startup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dmoriart) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
