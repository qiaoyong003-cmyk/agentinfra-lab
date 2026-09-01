# AgentInfra Lab

**LLM Reliability · Agent Engineering ·Linux / Private AI / Xinchuang**

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

## Research Cases

### #01 · Tool Calling Silent Failure

A real failure observed during Coding Agent integration testing:

- HTTP 200
- SDK returned normally
- `finish_reason = tool_calls`
- but actual `tool_calls = 0`

**2×2 protocol × streaming test**

| Protocol | Non-Streaming | Streaming |
|---|---|---|
| Anthropic | ✅ | ✅ |
| OpenAI Compatible | ✅ | ❌ |

**Engineering rule**

> Declared Tool Calling + Actual Tool Call = 0 → Hard Fail

**Key lesson**

> Don't assume it works. Prove it.

👉 **[Read the full case →](./cases/01-tool-calling-silent-failure.md)**

### #02 · Hidden Context Overhead

A minimal LLM request containing only `hi` produced unexpectedly high input token usage.

- Explicit input: ≈ 8–10 tokens
- Anthropic path: `1300` input tokens
- OpenAI Compatible path: `1063` prompt tokens
- Difference between protocol paths: ≈ 237–240 tokens

**Engineering lesson**

> Don't assume what the model sees. Measure it.

👉 **[Read the full case →](./cases/02-hidden-context-overhead.md)**

### #03 · Context Budget Silent Failure

The context budget subsystem detected an invalid runtime state, but downstream execution could still continue unless that state was explicitly enforced.

- Context budget check: `over_budget_error = true`
- Downstream risk: Assembler may continue building the Context Bundle
- Reliability gap: error detection without control-flow enforcement
- Current fix: explicit `raise OverBudgetError(...)` hard fail

**Engineering lesson**

> Error detection must change control flow.

👉 **[Read the full case →](./cases/03-context-budget-silent-failure.md)**

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
