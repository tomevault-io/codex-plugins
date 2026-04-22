# Legado书源驯兽师 - AI Coding Agent Instructions

## Project Overview
This is an AI-driven tool for automatically generating book sources for the Legado reading Android app. The system analyzes website HTML structures and creates compliant JSON configurations using a knowledge-driven approach.

## Architecture
- **Skill Package**: `SKILL.md` - Core AI agent logic with 23 specialized tools
- **Debug Engine**: `debugger/` - Python-based testing framework simulating Legado's Kotlin parser
- **Knowledge Base**: `assets/` - 24.93MB of documentation, 1751 real book source examples
- **Legado Source**: `legado/` - Official app source code for compatibility validation

## Critical Workflows

### Testing Book Sources
```bash
# Test a book source with search keyword
python debugger/test_universal.py path/to/book_source.json -k "斗破苍穹"

# Enable auto-fix with max attempts
python debugger/test_universal.py path/to/book_source.json -k "keyword" --auto-fix
```

### Environment Setup
```bash
# Install dependencies
pip install -r debugger/requirements.txt

# Run in Trae IDE with MCP protocol support
# Skill package auto-loads from .trae/skills/
```

## Key Patterns & Conventions

### Knowledge-First Approach
**Always query knowledge base before rule generation:**
- `search_knowledge()` for CSS selectors, POST configs, regex patterns
- `get_css_selector_rules()` for complete selector syntax
- `get_real_book_source_examples()` for proven templates

### Rule Validation
- Strict Legado JSON schema compliance
- No forbidden fields: `textNodes`, `ownText`, `allElements`
- Regex patterns use `##` delimiter for replacement
- `nextContentUrl` requires pagination detection

### 5-Stage Development Workflow
1. **Collect Info**: Query knowledge, detect encoding, fetch HTML
2. **Review Rules**: Write rules, validate syntax, handle edge cases  
3. **Create Source**: Generate complete JSON, debug test
4. **Output JSON**: Save to `output/book_sources/`
5. **Self-Evolve**: Learn from successful patterns

### Encoding Handling
- Auto-detect charset from HTML/response headers
- Fallback: GBK → UTF-8 conversion for Chinese sites
- Explicit encoding specification in book source config

### Error Patterns
- XPath failures → fallback to CSS selectors
- Network timeouts → retry with different User-Agent
- Parse errors → validate HTML structure first

## Integration Points
- **Trae IDE**: MCP protocol for skill activation
- **Web Scraping**: requests + BeautifulSoup with encoding detection
- **JSON Schema**: Strict validation against Legado's BookSource model
- **External APIs**: None - fully self-contained analysis

## File Organization
- `config/agent_llm_config.json` - LLM parameters (doubao-seed-1-8-251228)
- `assets/knowledge_base/` - 1751 real source examples for pattern matching
- `debugger/engine/` - Core parsing logic ported from Kotlin
- `output/book_sources/` - Generated JSON outputs

## Quality Assurance
- 90%+ accuracy through real HTML analysis
- Automated testing with `test_universal.py`
- Knowledge base self-auditing for consistency
- Human intervention support for complex cases

## Common Pitfalls
- Skipping knowledge queries → invalid rules
- Using generic CSS selectors → parsing failures  
- Ignoring encoding detection → garbled text
- Manual JSON editing → schema violations</content>
<parameter name="filePath">d:\pack_project_1771468148809\legadoSkill\.github\copilot-instructions.md

---
> Source: [DandanLLab/legadoSkill](https://github.com/DandanLLab/legadoSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
