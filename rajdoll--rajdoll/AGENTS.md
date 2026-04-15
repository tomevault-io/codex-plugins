# Multi-Agent Security Testing System - Refactoring Instructions

## Current State Analysis
- 11 MCP servers exist (authentication-testing/, input-validation-testing/, etc.)
- multi_agent_system/ folder has partial agent implementation
- Architecture is currently chatbot-style, needs refactoring to true multi-agent

## Target Architecture

### 1. True Multi-Agent System
Each agent must be:
- **Autonomous**: Makes own decisions via LLM planning
- **Specialized**: Expert in ONE OWASP WSTG category only
- **Context-aware**: Reads shared_context from previous agents, writes findings for next agents
- **Independent**: Has own LLM instance with specialized system prompt

### 2. Key Components

**Agents (multi_agent_system/agents/):**
- Each agent has `execute(job_id, target, shared_context)` method
- Planning phase: Ask LLM "Given context X, what tests should I run?"
- Execution phase: Call MCP tools via mcp_client
- Context update: Write findings to shared_context for next agents
- Pattern:
```python
  async def execute(self, job_id, target, shared_context):
      # 1. Read context from previous agents
      tech_stack = shared_context.get('tech_stack')
      
      # 2. Ask LLM for test plan based on context
      plan = await self.llm_client.plan(context=shared_context)
      
      # 3. Execute tests via MCP
      for test in plan.tests:
          result = await self.mcp_client.call_tool(test.tool, test.params)
      
      # 4. Update shared_context for next agents
      shared_context.update({'credentials': found_creds})
```

**Orchestrator (multi_agent_system/orchestrator.py):**
- Coordinates agent execution order
- Manages shared_context (dict that grows as agents add findings)
- Phase 1: ReconAgent (always first)
- Phase 2: LLM strategic planning for optimal agent order
- Phase 3: Execute agents sequentially/parallel based on dependencies
- Phase 4: Generate report

**Shared Context Flow:**
```
shared_context = {}
→ ReconAgent adds: tech_stack, entry_points
→ AuthAgent reads tech_stack, adds: credentials
→ SessionAgent reads credentials, adds: session_info
→ SQLiAgent reads tech_stack + entry_points independently
```

**MCP Integration:**
- Keep existing MCP servers (authentication-testing/, etc.)
- Agents call MCP via mcp_client wrapper
- Each agent knows which MCP server to use for its domain

### 3. Anti-Stuck Mechanisms
Implement in base_agent.py:
- Circuit breaker (max 3 failures per tool)
- Timeout (5 min per tool call)
- Graceful degradation (skip failed tests, continue)
- Comprehensive logging

### 4. Web Interface
- FastAPI backend (api/main.py)
- Fire-and-forget: POST /api/scan → returns job_id immediately
- Background execution via Celery workers
- Real-time progress: WebSocket updates
- Report retrieval: GET /api/report/{job_id}

### 5. Key Differences from Current Code
**WRONG (current - chatbot style):**
- User chats → LLM decides tool → Execute
- Single conversation flow
- User-dependent

**CORRECT (target - multi-agent):**
- User submits → Orchestrator → 11 agents work autonomously
- Each agent has own AI brain with specialized prompt
- Context flows between agents
- User-independent (fire-and-forget)

## Code Generation Rules

1. **Agent System Prompts**: Must be domain-specific OWASP expert prompts, not generic
2. **Context Management**: Always read from AND write to shared_context
3. **LLM Usage**: Each agent asks LLM for adaptive test planning, not static checklists
4. **MCP Calls**: Use existing MCP servers, don't reimplement tools
5. **Logging**: Log every context read/write for debugging
6. **Error Handling**: Implement circuit breaker pattern, never let one failure stop entire scan

## Priority Order
1. Fix base_agent.py → proper context-aware execution pattern
2. Fix orchestrator.py → proper agent coordination with strategic planning
3. Update each agent → specialized prompts + context usage
4. Fix mcp_client.py → reliable MCP server communication
5. Add comprehensive logging → track context flow between agents

## Example Agent Structure
See reconnaissance_agent.py as reference - all other agents should follow this pattern:
- Specialized system prompt (OWASP expert)
- Context-aware planning (read shared_context)
- Tool execution (via MCP)
- Context updates (write findings)
- Proper error handling

When generating code, ensure it matches TRUE multi-agent architecture, not chatbot-style.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rajdoll) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
