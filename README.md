AgentInfra Lab
LLM Reliability · Agent Engineering ·Linux / Private AI / Xinchuang
研究如何让 AI Agent 真正进入企业生产环境。
不训练模型，研究怎么让模型真正干活。
Focus
LLM endpoint / gateway reliability
Coding Agent engineering
Tool Calling / Streaming reliability
Agent runtime failure detection
Linux / Windows / private deployment
Xinchuang / domestic AI environments
Long-term memory governance
Enterprise AI support systems
Research Cases
#01 · Tool Calling Silent Failure
A real failure observed during Coding Agent integration testing:
HTTP 200
SDK returned normally
finish_reason = tool_calls
but actual tool_calls = 0
2×2 protocol × streaming test
Protocol	Non-Streaming	Streaming
Anthropic	✅	✅
OpenAI Compatible	✅	❌
Engineering rule
> Declared Tool Calling + Actual Tool Call = 0 → Hard Fail
Key lesson
> Don't assume it works. Prove it.
👉 Read the full case →
#02 · Hidden Context Overhead
A minimal LLM request containing only hi produced unexpectedly high input token usage.
Explicit input: ≈ 8–10 tokens
Anthropic path: 1300 input tokens
OpenAI Compatible path: 1063 prompt tokens
Difference between protocol paths: ≈ 237–240 tokens
Engineering lesson
> Don't assume what the model sees. Measure it.
👉 Read the full case →
#03 · Context Budget Silent Failure
The context budget subsystem detected an invalid runtime state, but downstream execution could still continue unless that state was explicitly enforced.
Context budget check: over_budget_error = true
Downstream risk: Assembler may continue building the Context Bundle
Reliability gap: error detection without control-flow enforcement
Current fix: explicit raise OverBudgetError(...) hard fail
Engineering lesson
> Error detection must change control flow.
👉 Read the full case →
#04 · Real Database Type Boundary Failure
A real failure observed when running the application against PostgreSQL:
Mock-based tests passed
The visible ID value was identical
Application input type was str
ORM return type was uuid.UUID
Valid owner checks failed and export filters could return zero records
Mock × real dependency comparison
Validation Path	Mocked Tests	Real PostgreSQL
Runtime ID type	str	uuid.UUID
Owner comparison	✅	❌
Export / serialization path	✅	❌
Root fix
> Native PostgreSQL UUID + Consistent Python-side str contract
The shared model alias was applied across 45 UUID column declarations. The historical post-fix validation record was 58 passed, with no database migration required.
Engineering rule
> Mocks must preserve boundary behavior, not only example values.
Key lesson
> Green tests are evidence. The business result is the acceptance criterion.
👉 Read the full case →
#05 · Multi-Agent Code Review
A parallel review of the NeverReset backend expanded the inspection surface:
5 review Agents
107 backend files
approximately 8,025 lines
142 findings: 24 severe / 74 medium / 44 improvement
12 severe findings fixed immediately across 14 files
130 remaining findings registered
Finding × engineering state
Review output	Engineering meaning
Finding generated	Candidate issue
Evidence checked	Triaged issue
Fix applied	Remediation candidate
Regression passed	Verified fix
Not fixed, recorded	Tracked technical debt
Engineering rule
> Review Output ≠ Verified Defect
Key lesson
> AI can expand the review surface. It cannot replace verification.
👉 Read the full case →
#06 · AI Memory Confirmation Overload
A long-term memory system must decide when to save automatically and when to ask the user:
Confirming every memory creates a growing review burden
Saving everything can turn model inference into apparent fact
Explicit facts, model inferences, and important conclusions need different handling
Conflicting conclusions should remain visible until a decision is required
Information × handling policy
Information	Default handling
Explicit preference or stable fact	Save automatically
Model-derived inference	Preserve source and review state
Important project conclusion	Confirm when required
Conflicting conclusions	Keep both; request judgment when relevant
Engineering rule
> The confidence and impact of a memory determine its confirmation cost.
Key lesson
> Remember correctly. Ask only when judgment is required.
👉 Read the full case →
Existing Projects
llmc
A model-agnostic terminal Coding Agent with endpoint capability testing.
Key areas:
Tool Calling validation
Streaming capability detection
Provider capability declaration
MCP integration
Session persistence
Context compaction
Windows/Linux shell engineering
NeverReset
A long-term memory system for AI applications.
Key areas:
User-owned memory
Versioned memory records
Conflict surfacing
Memory governance
Context assembly
Cross-model architecture design
Industrial AI Support Assistant
A deployed AI support assistant for industrial data-transfer / network isolation products.
Key areas:
AI + domain knowledge
Case memory
Human handoff
Source-code-assisted diagnosis
Field support workflows
Principle
Evidence over assumptions.
Real tests, real failures, explicit boundaries.
