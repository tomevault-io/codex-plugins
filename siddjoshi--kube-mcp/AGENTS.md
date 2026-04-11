# Copilot Instructions for Kubernetes MCP Server

## Project Overview
This is a **Model Context Protocol (MCP) server** that bridges AI assistants with Kubernetes clusters, providing 50+ tools for cluster management through the MCP standard. The server supports three transport modes: stdio (default), SSE, and HTTP chunked streaming for real-time operations.

## Key Architecture Patterns

### Manager-Based Architecture
The codebase follows a manager pattern where each domain has its own manager class:
- `KubernetesManager` - Core K8s API client management and connection handling
- `ToolManager` - Registers and executes all 50+ MCP tools (pods, deployments, helm, etc.)
- `ResourceManager` - Handles MCP resource discovery and hierarchical organization
- `PromptManager` - Provides guided troubleshooting workflows (12+ diagnostic prompts)
- `SecurityManager` - RBAC validation and audit logging
- `MetricsManager` - Prometheus metrics and health monitoring

### Configuration with Zod Validation
All configuration uses Zod schemas in `src/config/config.ts`. The `loadConfig()` function loads from environment variables with intelligent defaults. **Always use the Config type for type safety.**

### Kubernetes Authentication Strategy
`KubernetesManager.loadKubeConfig()` implements a waterfall authentication approach:
1. In-cluster service account (`/var/run/secrets/kubernetes.io/serviceaccount`)
2. `KUBECONFIG_YAML` environment variable (inline YAML)
3. `KUBECONFIG_JSON` environment variable (inline JSON)
4. Minimal config (`K8S_SERVER` + `K8S_TOKEN`)
5. Custom path (`KUBECONFIG_PATH`)
6. Default `~/.kube/config`

### Transport Architecture
Three distinct transport modes with different initialization patterns:
- **stdio**: Standard MCP transport for AI assistants like Claude Desktop
- **sse**: Server-sent events with Express endpoint  
- **http-chunked**: Custom streaming endpoint at `/call-tool-chunked` for real-time progress

## Development Workflows

### Build and Development Commands
```bash
npm run build        # TypeScript compilation to dist/
npm run dev          # Watch mode with tsc --watch
npm start           # Run compiled server
npm run start:direct # Run directly with ts-node (development)
npm test            # Jest tests with 95% coverage requirement
npm run lint        # ESLint with TypeScript rules
```

### Transport Testing Strategies
- **Local Development**: `npm run start:direct` with stdio transport
- **HTTP Testing**: `MCP_TRANSPORT=http-chunked npm start` then POST to `/call-tool-chunked`
- **Container Testing**: `docker build -t mcp-server . && docker run -p 3000:3000 mcp-server`
- **Claude Desktop**: Add to `claude_desktop_config.json` with stdio transport

### Kubernetes Deployment Pattern
Uses `k8s-mcp-server.yaml` with ServiceAccount, RBAC, Deployment, and Service. The container runs with in-cluster authentication and `http-chunked` transport by default.

## Code Conventions

### Error Handling Pattern
Use the established error handling utilities:
```typescript
import { logger, logError, logOperation } from '../utils/logger.js';

try {
  const result = await someK8sOperation();
  logOperation('operation_name', additionalContext);
  return result;
} catch (error) {
  logError('Failed to perform operation', error as Error);
  throw this.k8sManager.handleKubernetesError(error);
}
```

### Tool Registration Pattern
New tools in `ToolManager` follow this pattern:
```typescript
private createNewToolHandler(): ToolHandler {
  return {
    name: 'tool-name',
    description: 'Clear description for AI agents',
    inputSchema: {
      type: 'object',
      properties: { /* Zod-style schema */ },
      required: ['requiredParam']
    },
    handler: async (args) => {
      // Validate, execute, return ToolResult
    }
  };
}
```

### KubectlExecutor Pattern
Tools use `KubectlExecutor` to translate commands to K8s API calls rather than spawning kubectl processes:
```typescript
const result = await this.kubectlExecutor.execute('get', ['pods', '-n', namespace]);
return {
  content: [{ type: 'text', text: result.success ? result.output : result.error }],
  isError: !result.success
};
```

### Prompt System Architecture
`PromptManager` provides 12+ diagnostic workflows with dynamic arguments:
- `k8s-pod-crashloop-diagnose` - CrashLoopBackOff analysis
- `k8s-rbac-access-denied` - RBAC troubleshooting
- `k8s-deployment-rollout-troubleshoot` - Rollout issues
Register new prompts in `registerPrompts()` with structured arguments and guided workflows.

### ES Module Imports
**Critical**: All imports must use `.js` extensions for compiled output:
```typescript
import { KubernetesManager } from '../kubernetes/manager.js';  // ✅ Correct
import { KubernetesManager } from '../kubernetes/manager';     // ❌ Wrong
```

## Integration Points

### Kubernetes Client Libraries
Uses `@kubernetes/client-node` with multiple API clients initialized in `KubernetesManager`:
- `coreV1Api` - Pods, Services, Namespaces
- `appsV1Api` - Deployments, StatefulSets
- `customObjectsApi` - CRDs and custom resources
- Proper connection testing and health checks implemented

### MCP Protocol Integration
Server implements all MCP capabilities:
- **Tools**: Kubernetes operations exposed as MCP tools
- **Resources**: Dynamic K8s resource discovery with hierarchical URIs
- **Prompts**: Guided troubleshooting workflows
- Request handlers in `server.ts` include security validation and metrics

### HTTP Chunked Streaming
Special transport mode for real-time tool execution with progress updates:
```typescript
// POST to /call-tool-chunked with {"name": "tool", "args": {...}}
// Returns streaming JSON responses with progress and final result
```

### Security and RBAC
`SecurityManager` validates operations against K8s RBAC before execution. Use `allowOnlyNonDestructiveTools` config for read-only mode.

## Key Files for Reference
- `src/server.ts` - Main MCP server setup and request routing
- `src/kubernetes/manager.ts` - K8s connection and API client management
- `src/tools/manager.ts` - All 50+ tool implementations
- `src/config/config.ts` - Comprehensive Zod configuration schema
- `package.json` - Build scripts and strict TypeScript/Jest setup

## Common Pitfalls
- Missing `.js` extensions in imports will break compilation
- Not using `ensureInitialized()` before K8s operations
- Forgetting to register new tools in `ToolManager.registerTools()`
- Not handling Kubernetes errors through `handleKubernetesError()`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/siddjoshi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/siddjoshi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
