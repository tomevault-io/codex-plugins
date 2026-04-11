# Copilot Instructions for Neural Network Project

This repository contains a Python-based implementation of a Neural Network from scratch, primarily using NumPy.

## Project Architecture
- **Core Logic**: The entire neural network implementation resides in `main.py`.
- **Class Structure**: The `NeuralNetwork` class encapsulates the model state (weights, biases) and behavior (`feedforward`, `backpropagate`, `train`).
- **Math**: Activation functions (Sigmoid) and their derivatives are defined as standalone functions.
- **Dependencies**: Relies heavily on `numpy` for vectorized matrix operations.

## Developer Workflow
- **Running the Code**: Execute the script directly: `python main.py`.
- **Training**: The `__main__` block handles data setup (XOR problem), network initialization, and the training loop.
- **Performance**: Execution time is measured using the `time` module within the script.

## Coding Conventions
- **Vectorization**: Prefer NumPy array operations over explicit loops for performance (e.g., `np.dot`, `np.sum`).
- **Structure**: Keep the model definition and execution logic in `main.py` unless refactoring is requested.
- **Entry Point**: Always use `if __name__ == '__main__':` to guard the execution logic (training/testing).
- **Documentation**: Add comments explaining the mathematical operations (e.g., derivatives, error calculations).

## Key Files
- `main.py`: Contains the `NeuralNetwork` class, helper functions, and the training/testing loop.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cscottgit)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/cscottgit)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
