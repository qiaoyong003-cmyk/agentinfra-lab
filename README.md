# AgentInfra Lab

**LLM Reliability · Agent Engineering · Linux / Xinchuang**

研究如何让 AI Agent 真正进入企业生产环境。

不训练模型，研究怎么让模型真正干活。

## Focus

- LLM endpoint / gateway reliability
- Coding Agent engineering
- Tool Calling / Streaming reliability
- Agent runtime failure detection
- Linux / Windows / private deployment
- Xinchuang / domestic AI environments
- Long-term memory governance
- Enterprise AI support systems

## Current Research

### #01 Tool Calling Silent Failure

A real failure observed during Coding Agent integration testing:

- HTTP 200
- SDK returned normally
- `finish_reason = tool_calls`
- but actual `tool_calls = 0`

2×2 test:

| Protocol | Non-Streaming | Streaming |
|---|---|---|
| Anthropic | ✅ | ✅ |
| OpenAI Compatible | ✅ | ❌ |

Engineering rule:

> Declared Tool Calling + Actual Tool Call = 0 → Hard Fail

The key lesson:

> Don't assume it works. Prove it.

## Existing Projects

### llmc
A model-agnostic terminal Coding Agent with endpoint capability testing.

Key areas:
- Tool Calling validation
- Streaming capability detection
- Provider capability declaration
- MCP integration
- Session persistence
- Context compaction
- Windows/Linux shell engineering

### NeverReset
A long-term memory system for AI applications.

Key areas:
- User-owned memory
- Versioned memory records
- Conflict surfacing
- Memory governance
- Context assembly
- Cross-model architecture design

### Industrial AI Support Assistant
A deployed AI support assistant for industrial data-transfer / network isolation products.

Key areas:
- AI + domain knowledge
- Case memory
- Human handoff
- Source-code-assisted diagnosis
- Field support workflows

## Principle

**Evidence over assumptions.**

Real tests, real failures, explicit boundaries.
