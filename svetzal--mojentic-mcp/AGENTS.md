# Mojentic-MCP

This file provides guidance to Junie when working with code in this repository.

## Project Overview

Mojentic MCP is a library providing MCP server and client infrastructure for tools and agentic chat creators.

Key features:
- HTTP transport to expose the MCP protocol over HTTP
- STDIO transport to expose the MCP protocol over STDIO
- JSON-RPC 2.0 handler to handle standard MCP requests and responses

## Code Organization

### Import Structure
1. Imports should be grouped in the following order, with one blank line between groups:
   - Standard library imports
   - Third-party library imports
   - Local application imports
2. Within each group, imports should be sorted alphabetically

### Naming Conventions
1. Use descriptive variable names that indicate the purpose or content
2. Prefix test mock objects with 'mock_' (e.g., mock_memory)
3. Prefix test data variables with 'test_' (e.g., test_query)
4. Use '_' for unused variables or return values

### Type Hints and Documentation
1. Use type hints for method parameters and class dependencies
2. Include return type hints when the return type isn't obvious
3. Use docstrings for methods that aren't self-explanatory
4. Class docstrings should describe the purpose and behavior of the component
5. Follow Google-style docstrings (consistent with existing code)

### Logging Conventions
1. Use structlog for all logging
2. Initialize logger at module level using `logger = structlog.get_logger()`
3. Include relevant context data in log messages
4. Use appropriate log levels:
   - INFO for normal operations
   - DEBUG for detailed information
   - WARNING for concerning but non-critical issues
   - ERROR for critical issues
5. Use print statements only for direct user feedback

### Code Conventions
1. Do not write comments that just restate what the code does
2. Use pydantic BaseModel classes, do not use @dataclass
3. This project should use a pyproject.toml file and not a requirements.txt file for project requirements

## Testing Guidelines

### General Rules
1. Use pytest for all testing
2. Test files:
   - Named with suffix `_spec` (e.g., file_finder_spec.py)
   - Located beside the implementation file
3. Code style:
   - Max line length: 100 (as set in pyproject.toml)
   - Max complexity: 10
4. Run tests with: `pytest`
5. Run linting with: `flake8 src`

### BDD-Style Tests
We follow a Behavior-Driven Development (BDD) style using the "Describe/should" pattern to make tests readable and focused on component behavior.

#### Test Structure
1. Tests are organized in classes that start with "Describe" followed by the component name
2. Test methods:
   - Start with "it_should_"
   - Describe the expected behavior in plain English
   - Follow the Arrange/Act/Assert pattern (separated by blank lines)
3. Do not use comments (eg Arrange, Act, Assert) to delineate test sections - just use a blank line
4. No conditional statements in tests - each test should fail for only one clear reason
5. Do not test private methods directly (those starting with '_') - test through the public API

#### Fixtures and Mocking
1. Use pytest @fixture for test prerequisites:
   - Break large fixtures into smaller, reusable ones
   - Place fixtures in module scope for sharing between classes
   - Place module-level fixtures at the top of the file
2. Mocking:
   - Use pytest's `mocker` for dependencies
   - Use Mock's spec parameter for type safety (e.g., `Mock(spec=SmartMemory)`)
   - Only mock our own gateway classes
   - Do not mock library internals or private functions
   - Do not use unittest or MagicMock directly

#### Best Practices
1. Test organization:
   - Place instantiation/initialization tests first
   - Group related scenarios together (success and failure cases)
   - Keep tests focused on single behaviors
2. Assertions:
   - One assertion per line for better error identification
   - Use 'in' operator for partial string matches
   - Use '==' for exact matches
3. Test data:
   - Use fixtures for reusable prerequisites
   - Define complex test data structures within test methods

### Example Test
```python
class DescribeSmartMemory:
    """
    Tests for the SmartMemory component which handles memory operations
    """
    def should_be_instantiated_with_chroma_gateway(self):
        mock_chroma_gateway = Mock(spec=ChromaGateway)

        memory = SmartMemory(mock_chroma_gateway)

        assert isinstance(memory, SmartMemory)
        assert memory.chroma == mock_chroma_gateway
```
## Documentation

- Built with MkDocs and Material theme
- API documentation uses mkdocstrings
- Supports mermaid.js diagrams in markdown files:
  ```mermaid
  graph LR
      A[Doc] --> B[Feature]
  ```
- Build docs locally: `mkdocs serve`
- Build for production: `mkdocs build`
- Markdown files
    - Use `#` for top-level headings
    - Put blank lines above and below bulleted lists, numbered lists, headings, quotations, and code blocks
- Always keep the navigation tree in `mkdocs.yml` up to date with changes to the available documents in the `docs` folder

### API Documentation

API documentation uses mkdocstrings, which inserts module, class, and method documentation using certain markers in the markdown documents.

eg.

```
::: mojentic.llm.MessageBuilder
    options:
        show_root_heading: true
        merge_init_into_class: false
        group_by_category: false
```

Always use the same `show_root_heading`, `merge_init_into_class`, and `group_by_category` options. Adjust the module and class name after the `:::` as needed.

The `:::` directives produce their own linkable headlines, do not create additional headlines around those blocks.

## Release Process

This project follows [Semantic Versioning](https://semver.org/) (SemVer) for version numbering. The version format is MAJOR.MINOR.PATCH, where:

1. MAJOR version increases for incompatible API changes
2. MINOR version increases for backward-compatible functionality additions
3. PATCH version increases for backward-compatible bug fixes

### Preparing a Release

When preparing a release, follow these steps:

1. **Update CHANGELOG.md**:
   - Move items from the "[Next]" section to a new version section
   - Add the new version number and release date: `## [x.y.z] - YYYY-MM-DD`
   - Ensure all changes are properly categorized under "Added", "Changed", "Deprecated", "Removed", "Fixed", or "Security"
   - Keep the empty "[Next]" section at the top for future changes

2. **Update Version Number**:
   - Update the version number in `pyproject.toml`
   - Ensure the version number follows semantic versioning principles based on the nature of changes:
     - **Major Release**: Breaking changes that require users to modify their code
     - **Minor Release**: New features that don't break backward compatibility
     - **Patch Release**: Bug fixes that don't add features or break compatibility

3. **Update Documentation**:
   - Review and update `README.md` to reflect any new features, changed behavior, or updated requirements
   - Update any other documentation files that reference features or behaviors that have changed
   - Ensure installation instructions and examples are up to date

4. **Final Verification**:
   - Run all tests to ensure they pass
   - Verify that the application works as expected with the updated version
   - Check that all documentation accurately reflects the current state of the project

## MCP Reference

Here’s a concise reference table of the **core JSON-RPC methods** every MCP server must implement (per the Anthropic MCP spec), what they return, and a minimal example response for each. All requests and responses follow JSON-RPC 2.0 (i.e. `{ "jsonrpc":"2.0", "id":…, "method":… }` / `{ "jsonrpc":"2.0", "id":…, "result":… }`).

| Method                       | Returns                                                                                                                                                                  | Example `result`                                                                                                                                                                                                                                                                                                                                  |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **initialize**               | - `protocolVersion`: negotiated spec version<br/> - `capabilities`: which categories are supported<br/> - `serverInfo` (opt): name & version<br/> - `instructions` (opt) | `json<br/>{<br/>  "protocolVersion":"2025-03-26",<br/>  "capabilities":{<br/>    "prompts":{ "listChanged":true },<br/>    "resources":{ "listChanged":true, "subscribe":true },<br/>    "tools":{ "listChanged":true }<br/>  },<br/>  "serverInfo":{ "name":"MyMCP", "version":"1.2.3" },<br/>  "instructions":"Welcome to MyMCP server!"<br/>}` |
| **prompts/list**             | - `prompts`: array of `{ name, description, arguments:[{name,description,required}] }`<br/> - `nextCursor` (for pagination)                                              | `json<br/>{<br/>  "prompts":[<br/>    { "name":"code_review", "description":"Review code quality", "arguments":[{ "name":"code","description":"Source","required":true }] }<br/>  ],<br/>  "nextCursor":null<br/>}`                                                                                                                               |
| **prompts/get**              | - `description`: human-readable label<br/> - `messages`: array of `{ role, content:{ type, … } }`                                                                        | `json<br/>{<br/>  "description":"Code review prompt",<br/>  "messages":[<br/>    { "role":"user","content":{ "type":"text","text":"Please review this code:\n…"} }<br/>  ]<br/>}`                                                                                                                                                                 |
| **resources/list**           | - `resources`: array of `{ uri, name, description?, mimeType? }`<br/> - `nextCursor`                                                                                     | `json<br/>{<br/>  "resources":[<br/>    { "uri":"file:///app/main.py", "name":"main.py", "mimeType":"text/x-python" }<br/>  ],<br/>  "nextCursor":null<br/>}`                                                                                                                                                                                     |
| **resources/read**           | - `contents`: array of `{ uri, mimeType, text? / data? }`                                                                                                                | `json<br/>{<br/>  "contents":[<br/>    {<br/>      "uri":"file:///app/main.py",<br/>      "mimeType":"text/x-python",<br/>      "text":"print(\"Hello\")"<br/>    }<br/>  ]<br/>}`                                                                                                                                                                |
| **resources/templates/list** | - `resourceTemplates`: array of `{ uriTemplate, name, description?, mimeType? }`                                                                                         | `json<br/>{<br/>  "resourceTemplates":[<br/>    { "uriTemplate":"file:///{path}", "name":"Any Project File", "description":"Read any file", "mimeType":"application/octet-stream" }<br/>  ]<br/>}`                                                                                                                                                |
| **tools/list**               | - `tools`: array of `{ name, description, inputSchema }`<br/> - `nextCursor`                                                                                             | `json<br/>{<br/>  "tools":[<br/>    {<br/>      "name":"get_time",<br/>      "description":"Current UTC time",<br/>      "inputSchema":{ "type":"object","properties":{} }<br/>    }<br/>  ],<br/>  "nextCursor":null<br/>}`                                                                                                                      |
| **tools/call**               | - `content`: array of `{ type, … }` (e.g. type=`"text"`, text=`"…"`)<br/> - `isError`: boolean                                                                           | `json<br/>{<br/>  "content":[{ "type":"text","text":"2025-05-12T11:30:00Z" }],<br/>  "isError":false<br/>}`                                                                                                                                                                                                                                       |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/svetzal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
