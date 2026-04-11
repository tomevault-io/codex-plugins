# FraudNet Copilot Instructions

## Project Overview
FraudNet is a deep learning neural network built in Rust for detecting Fraud, Waste, and Abuse. The trained model is compiled to ONNX runtime for client-side use in web browsers.

## Technology Stack
- Primary language: Rust
- Machine Learning: Deep learning neural network
- Output format: ONNX runtime compatible
- Target deployment: Client-side (web browser)

## Code Style and Conventions
- Follow Rust best practices and idioms
- Use `cargo fmt` for code formatting
- Use `cargo clippy` for linting
- Write comprehensive documentation comments for public APIs
- Prefer safe Rust code; use unsafe blocks only when absolutely necessary with clear justification

## Neural Network Architecture
The project implements a multi-layer neural network with:
- Input layer: 3 features (after UMAP dimensionality reduction)
- Hidden layers: 9 layers with progressively decreasing neuron counts (64 → 52 → 42 → 32 → 26 → 22 → 20 → 16 → 8)
- Activation function: ReLU for hidden layers
- Output layer: 1 neuron with Sigmoid activation for binary classification

## Development Guidelines
- Ensure all neural network implementations are compatible with ONNX export
- Optimize for both training performance and inference speed
- Consider memory efficiency for client-side deployment
- Write unit tests for mathematical operations and network components
- Document any assumptions about input data preprocessing (especially UMAP transformation)

## Build and Test Commands
- Build: `cargo build`
- Run tests: `cargo test`
- Format code: `cargo fmt`
- Lint: `cargo clippy`
- Build for release: `cargo build --release`

## Security Considerations
- Validate all input data to prevent injection attacks
- Ensure model inference cannot be exploited
- Keep dependencies up to date for security patches
- Review any unsafe code blocks carefully

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johncoopertr)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/johncoopertr)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
