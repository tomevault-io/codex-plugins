# LeetCode Solutions Codebase Guide

## Project Structure & Organization
This is a LeetCode practice repository organized by difficulty level under `python/`:
- `easy/` - Fundamental algorithms and data structures
- `medium/` - More complex algorithms requiring multiple approaches
- `hard/` - Advanced algorithmic challenges

## Code Patterns & Conventions

### Solution Class Structure
Most medium/hard problems follow the LeetCode class-based pattern:
```python
class Solution:
    def problemMethod(self, params) -> return_type:
        # Implementation here
```

### Standalone Functions for Simple Problems
Easy problems and utility functions often use standalone function approach:
```python
def functionName(params) -> return_type:
    # Implementation here
```

### Import Conventions
- Always use `from typing import List` for type hints
- Common imports: `collections.deque`, `collections.defaultdict`, `functools.cache`
- External libraries like `matplotlib.pyplot` only for visualization (see `water-trap.py`)

### Testing & Execution Pattern
Every file includes a `if __name__ == "__main__":` block with:
- Multiple test cases (often commented out for quick testing)
- Example usage demonstrating the solution
- Expected output in comments when helpful

### Algorithm Implementation Styles

#### Graph Problems
- Use adjacency lists with `collections.defaultdict(list)`
- BFS with `collections.deque` for shortest paths
- Explicit `visited` sets or boolean matrices
- See `findifPathExistsInGraph.py`, `numberIslands.py`

#### Dynamic Programming
- Recursive solutions with `@functools.cache` decorator
- Bottom-up approaches for optimization
- See `coinChange.py`, `checkIfPathHasEqualNumberOfZeros.py`

#### Two Pointer Technique
- Common for array problems and optimization
- See `water-trap.py`, `shortestSnsortedContinuousSubarray.py`

#### Stack-Based Solutions
- Monotonic stacks for "next greater element" patterns
- See `monotonicStackExample.py`, `decodeStrint.py`

### Performance Considerations
- Include time/space complexity analysis in comments (see `numberIslands.py`)
- Provide both optimized and unoptimized solutions when educational (see `jumpGame2.py`)
- Use bitwise operations for optimization (see `checkIfPathHasEqualNumberOfZeros.py`)

### Naming Conventions
- Descriptive variable names: `left_max`, `right_max`, `is_closed`
- Helper method names: `bfs`, `dfs`, `recursionHelper`
- Direction arrays: `dirx`, `diry` for coordinate movement

### Debug & Development Workflow
- Use print statements for debugging (see examples in `findifPathExistsInGraph.py`)
- Comment out test cases to focus on specific scenarios
- Include edge cases in test blocks

When working on this codebase, prioritize clarity and educational value over brevity. Each solution should demonstrate the core algorithmic pattern clearly with appropriate comments explaining the approach.

---
> Source: [QuantumKuba/leetcodes](https://github.com/QuantumKuba/leetcodes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-11 -->
