
# URLPatternList Copilot Instructions

## Overview

This library implements a `URLPattern` collection called a `URLPatternList` that
has optimized matching against the set of patterns using a **prefix tree (trie)
data structure** instead of linear scanning.

`URLPattern` is a web standard that is also available in server runtimes like
Node and Deno.
- URLPattern on MDN:
  https://developer.mozilla.org/en-US/docs/Web/API/URL_Pattern_API
- Specification: https://urlpattern.spec.whatwg.org/

`URLPattern` is often used for client-side and HTTP-server routing. Especially
in the server routing use case, a server might have hundreds or thousands of
routes and need to match an incoming request URL to a route as fast as possible.
Linearly scanning a list of URLPattern and testing each pattern does not scale
well enough, and prefix-tree (trie) based search can be much faster.

## API

`URLPatternList` has two main APIs:
- `addPattern(pattern)` which adds a new pattern to the list and updates the
  internal prefix tree.
- `match(url, baseUrl)` which searches the prefix tree for a match.

Patterns cannot currently be removed or inserted into the middle of the list.
This may be an important feature to add in the future.

## Architecture

### Parsing

`src/lib/parse-pattern.ts` contains the `parse()` function which parses a
`URLPattern` into an array of `Parts`. The Part types include:
- Fixed: A fixed string, like `/api`.
- SegmentWildcard: A named parameter like `/:id`
- FullWildcard: A `*` wildcard
- Regex: A regex like `(\\d+)` or `(small|large)`

These parsed parts correspond to the types of prefix tree nodes as well. Fixed
parts are split by path separators (`/`) to make construction of the tree
simpler. **This splitting happens for all URL components, not just pathnames.**

The decision to split by `/` in search and hash components is intentional and
optimizes for common real-world patterns where path-like structures appear in
these components (e.g., `?path=/api/users` or `#/admin/dashboard`). This allows
the prefix tree to share common prefixes like `/api` even within search
parameters and hash fragments, improving performance.

Instead of building a radix-tree that sometimes needs to split nodes when new
nodes are added, we always make a path segment a separate node, and never
subdivide segments, so we only have to append to the tree. One of the goals of
this approach is to make it less costly to add patterns.

### Tree Construction

`URLPatternList.prototype.addPattern` (in `src/index.ts`) builds the prefix tree
from new patterns. The tree is made of `PrefixTreeNode` objects.

`PrefixTreeNode` is an abstract base class with five concrete implementations:
- `RooPrefixTreeNode`: The root of the tree.
- `FixedPrefixTreeNode`: Fixed string segments like `/api` or `/users`
- `WildcardPrefixTreeNode`: Named parameters like `:id` or `:userId` (with
  prefix/suffix support)
- `FullWildcardPrefixTreeNode`: Catch-all segments like `*`
- `RegexPrefixTreeNode`: Regex patterns like `(\\d+)` or `(small|large)`

`addPattern()` parses the pathname into a `Array<Part>` and then calls
`#addPatternToTree()` to recursively build the tree. During the recursion the
current part is checked against the current tree node to see if the tree node
matches that part.

Each tree node has it's own part matching logic. Fixed nodes match fixed parts
if their value and modifier are the same. Wildcard nodes match if their
modifier, prefix, and suffix are the same, and don't care about the parameter
name. Full wildcard nodes match if their modifier is the same. Regex nodes match
if the regex string is the same.

The point of the part matching logic is to share the ability to match a segment
of a URL pathname and consolidate and reduce the number of checks against the
path. This is why the parameter name doesn't matter for wildcard nodes.

If a tree node matches the current parsed part, then the process continues
recursively. If there is no match at that level of the tree, then a new tree
node is created and the process continues recursively from the new node.

Once all the parsed parts have been consumed the pattern is added to the final
tree node's `patterns` array, as part of a `URLPatternListItem` object.

`URLPatternListItem` stores the pattern, the value associated with the pattern
that was passed to `addPattern()`, and a `sequence` number that records the
order the pattern was added to the tree. `sequence` helps us implement the same
first-added-match-wins semantics of a linear search.

`PrefixTreeNode` has a `minSequence` field that tracks the minium `sequence`
number of any pattern stored in its subtree. This number is updated any time a
new pattern is added to a node's subtree. It's used to make the first-match
behavior efficient.

### URL Matching

Matching against a URL is done by calling `match()` on the root node, which
recursively calls `match()` on the subtrees.

`match()` is called with an array of `URLComponent`s. The given URL transformed
into this array so we can match it against the prefix tree from left-most
component (protocol) to right-most (hash) by incrementing an array index.

`match()` is implemented by each tree node implementation. It first checks for a
match at that node, A fixed tree node checks if the URL at the current
position starts with the node value, etc. If a node doesn't match, it's subtree
is not visited and the node's successor is checked. If a node does match, then
its patterns and children are checked. A node's patterns are only checked if the
entire URL string has been consumed, indicating that we should have found a
pattern match. A node's children are checked after that.

In order to implement first-match semantics, tree node children are sometimes
checked even after a match is found. The tree node match result is the whole
`URLPatternListItem`, which includes the `sequence` number. If a successor tree
node has a `minSequence` that is lower than the current best match's `sequence`,
then that node could contain a match for a pattern that was added _before_ the
current best match, and must be checked. But if the tree node's `minSequence` is
greater than the current best match's `sequence`, then the node can't possibly
have a pattern that should take precedence over the current best match and
should be skipped.

Searching a tree node's patterns and children is a common operation for all node
types and so implemented in `PrefixTreeNode.prototype.tryPatternsAndChildren()`.

## Development Workflows

This repo uses Wireit to coordinate and cache scripts. Wireit is transparent to
script runners, you use `npm` commands as usual. The important thing to know is
that Wireit runs script dependencies automatically - you don't have to run the
build script manually before tests.

* Build:
  ```bash
  npm run build
  ```
* Test:
  ```sh
  npm test
  ```
* Benchmarks:
  ```sh
  npm run benchmark
  ```

After major changes, run benchmarks to ensure no performance regressions.

When generating temporary debug scripts that aren't included in the build, you
do need to run the build first if the implementation has changed.

## Code Style

- Use very modern TypeScript: TypeScript 5.9 and JavaScript ES2024
- Always use ESM module syntax and never CommonJS
- Indent code 2 spaces
- Wrap code and comments at 80 columns
- Use arrow functions instead of the `function` keyword
- Use `undefined` for missing values instead of `null`, except when emulating
  DOM APIs that return `null`
- Use `const` everywhere possible, and `let` for mutable variables
- Always use `===` instead of `==` for comparisons
- Avoid boolean conversions and truthiness checks. Use explicit comparisons,
  like `if (x === undefined) {}`
- Try to avoid unnecessary object and string allocations. Use the `position`
  argument to string methods like `indexOf()` and `startsWith()`, and track
  current positions in our own methods to avoid calling `str.slice()`
- Use standard private fields, like `#foo`
- Prefer top-level functions for utilities instead of static class methods
- Keep tests as small as possible and focused on single scenarios
- Avoid assertion messages
- Test files are in `src/test` and end with `_test.ts`

## Testing

Tests are written with Node's `node:test` and `node:assert` packages, and run
with `node --test` as configured in the `wireit.build` object in `package.json`.

### Test Philosophy

- **Dual validation**: Every test runs against both optimized (prefix tree) and
  naive (linear) implementations
- **URLPattern compliance**: Uses `assertURLPatternBehavior()` to ensure
  identical behavior to native URLPattern
- **Performance validation**: Benchmarks verify 2-26x speedup depending on
  pattern count

## Key Files
- `src/index.ts`: Core prefix tree implementation and URLPatternList class
- `src/lib/parse-pattern.ts`: URLPattern parsing and Part type definitions  
- `src/test/naive-url-pattern-list.ts`: Reference linear implementation for
  validation
- `src/benchmark/benchmark.ts`: Performance testing with realistic pattern
  distributions

## Debugging Tips
- Use `list._treeRoot` to inspect tree structure during development
- Benchmark against naive implementation to verify performance gains

---
> Source: [justinfagnani/url-pattern-list](https://github.com/justinfagnani/url-pattern-list) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-07 -->
