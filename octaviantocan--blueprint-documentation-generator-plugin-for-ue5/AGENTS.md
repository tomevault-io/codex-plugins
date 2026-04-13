# UE Blueprint Docs Plugin - AI Coding Guidelines

## Project Overview

This is a UE5 Editor plugin that generates AI-powered documentation for Blueprint assets. It analyzes Blueprint graphs, extracts structured data, and uses Ollama API to create human-readable explanations and Markdown documentation.

**Specification Framework**: This project uses GitHub Spec Kit (github.com/github/spec-kit) for feature specifications, following their best practices and philosophy for clear, actionable, and stakeholder-focused requirements.

## Architecture & Data Flow

- **Entry Point**: `FBlueprintDocModule` handles UI integration and orchestrates processing
- **Analysis**: `FBlueprintTraverser` extracts functions, variables, and node connections from Blueprint graphs
- **AI Processing**: `FLLMExplanationGenerator` sends data chunks to Ollama API for explanations
- **Output**: `FDocumentationFileGenerator` converts JSON structures to Markdown files
- **Batch Processing**: `FBlueprintProcessingQueueManager` handles multiple Blueprints sequentially

## Key Conventions

### Code Organization

- **Single Responsibility**: Each class has one clear purpose (per constitution.md)
- **Comprehensive Documentation**: All code uses Doxygen-style headers with purpose, parameters, return values
- **Inline "Why" Comments**: Non-trivial logic includes `// why: explanation` comments
- **UE5 Standards**: Follows Epic's C++ standards with Blueprint exposure (`UCLASS`, `UPROPERTY`, `UFUNCTION`)

### Blueprint Analysis Patterns

- **Multiple Representations**: Functions extracted as hierarchical summaries, flow-based representations, and node details
- **Node Filtering**: Skip reroute nodes, comments, and documentation nodes during traversal
- **Connection Tracking**: Store input/output pin connections within `FBlueprintNodeDetails`

### AI Integration Patterns

- **Chunked Processing**: Large Blueprints split into manageable chunks for API calls
- **Queue Management**: Sequential processing with failure handling - stops entire queue on any failure
- **API Configuration**: Stored in plugin settings with Ollama endpoint and model

### File Structure Examples

```
Source/blueprintdoc/
├── Public/Core/           # Main module classes
├── Public/LLM/           # AI integration
├── Public/Documentation/ # Output generation
├── Private/              # Implementation details
└── BlueprintDoc.Build.cs # Module dependencies
```

## Development Workflow

### Prerequisites

- MUST consult Context7 documentation and OpenMemory knowledge graph before implementation
- MUST check Unreal Engine 5.0 documentation specifically before writing or speaking about UE5 code

### Specification Review

- **Double-check specs before coding**: Every time you begin implementation, review all specification documents (spec.md, plan.md, tasks.md, research.md, data-model.md, contracts/) to ensure complete understanding
- **Stop and ask for clarification**: If anything is unclear, missing, inconsistent, or could be specified better, STOP and ask the user to clarify rather than proceeding with assumptions
- **Specs first, code later**: Never begin coding until all specifications are complete, accurate, and approved by the user

### Building

- Built as UE5 plugin module through Unreal's build system
- Requires HTTP, JSON, BlueprintGraph, and Editor modules
- Output saved to `Plugins/blueprintdoc/Output/` folder

### Testing AI Features

- Set `bSkipLLM = true` in `FLLMExplanationGenerator` to test JSON generation without API calls
- Check `Response.txt` for API response debugging
- Use `UE_LOG` for debugging LLM interactions

### Adding New Node Types

1. Include appropriate K2Node headers in `BlueprintTraverser.cpp`
2. Add node type checks in traversal logic
3. Update `FBlueprintNodeDetails` if new connection types needed
4. Test with complex Blueprints containing the new node type

## Common Patterns

### Error Handling

```cpp
if (!Blueprint)
{
    UE_LOG(LogTemp, Error, TEXT("Invalid Blueprint provided"));
    return false;
}
```

### JSON Serialization

```cpp
TSharedPtr<FJsonObject> JsonObj = MakeShareable(new FJsonObject());
JsonObj->SetStringField(TEXT("name"), FunctionDetails.Name);
// Use FJsonObjectConverter for complex structs
```

### Async HTTP Requests

```cpp
TSharedRef<IHttpRequest> Request = FHttpModule::Get().CreateRequest();
Request->OnProcessRequestComplete().BindRaw(this, &FClass::OnRequestComplete);
Request->ProcessRequest();
```

### Blueprint Node Traversal

```cpp
for (UEdGraphNode* Node : Graph->Nodes)
{
    if (ShouldSkipNode(Node)) continue;
    // Process node connections and properties
}
```

## Integration Points

- **Content Browser**: Context menu extensions for "Generate Explanation" and "Generate JSON"
- **API Dependencies**: Ollama API (configurable endpoint/model)
- **UE Systems**: Relies on BlueprintGraph, KismetCompiler, and EditorScriptingUtilities

## Quality Gates

- All public APIs must be Blueprint-callable with proper metadata
- Memory management uses UE's garbage collection and smart pointers
- Performance: Forward declarations, UE containers, minimal allocations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OctavianTocan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/OctavianTocan)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
