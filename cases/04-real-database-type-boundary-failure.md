# \#04 Real Database Type Boundary Failure

> When a test double preserves the value but erases the type.

## Problem

A test suite can pass while the real application still fails at the database boundary.

In this case, the identifier looked the same everywhere:

```text
a1b2...
```

But the runtime values were not the same type:

```text
Mock / application input  → str
PostgreSQL ORM result     → uuid.UUID
```

That difference was enough to break owner checks, export filtering, JSON serialization, and several schema boundaries.

The central failure was:

> **Same visible value ≠ Same runtime type**

---

## Incident

The project lookup logic was intentionally simple:

```python
async def get_project(
    session: AsyncSession, *, project_id: str, owner_user_id: str
) -> Project:
    proj = await session.get(Project, project_id)

    if proj is None or proj.owner_user_id != owner_user_id:
        raise NotFoundError(f"project {project_id} not found")

    return proj
```

The application passed `owner_user_id` as a string.

With SQLAlchemy's default `Uuid()` behavior, PostgreSQL-backed ORM fields were returned as `uuid.UUID` objects.

So this comparison could become:

```python
uuid.UUID("550e8400-e29b-41d4-a716-446655440000")     != "550e8400-e29b-41d4-a716-446655440000"
```

The text representation is identical.

The Python objects are not.

The owner guard therefore rejected a valid owner and returned “project not found.”

This was a false-negative authorization check: a legitimate user could be denied access to their own object. It was **not** evidence of an authorization bypass or cross-user data exposure.

---

## Why the Tests Passed

The mock path represented identifiers as strings:

```text
Test fixture
↓
"550e8400-e29b-41d4-a716-446655440000"
↓
Application parameter: str
↓
Comparison succeeds
```

The real database path returned a different Python representation:

```text
PostgreSQL UUID column
↓
SQLAlchemy Uuid(as_uuid=True)
↓
uuid.UUID(...)
↓
Application parameter: str
↓
Comparison fails
```

The test double preserved the **value** but erased the **boundary behavior**.

That made the test internally consistent and externally inaccurate.

> **A mock can be logically correct inside the test and still misrepresent production.**

---

## Failure Pattern

The general pattern looks like this:

```text
Mocked dependency
↓
Returns convenient application-native type
↓
Tests pass
↓
Real dependency returns a different valid type
↓
Boundary assumption breaks
↓
Multiple downstream failures
```

Nothing was wrong with PostgreSQL returning a UUID.

Nothing was wrong with Python strings.

The failure was the absence of one explicit type contract between the persistence layer and the application layer.

This is a boundary failure, not an algorithm failure.

---

## Blast Radius

The same UUID-versus-string mismatch appeared across several paths.

| Path | Failure mode |
|---|---|
| Owner guards | A legitimate owner could be treated as a non-owner |
| Event payloads | UUID values could fail JSON serialization |
| Grounding models | ORM UUID objects crossed into fields defined as strings |
| Project export | Python-side UUID/string filters could return zero records |
| Tests and assertions | Results depended on which representation a fixture used |

The decision log records five classes of real product bugs with this shared root cause.

The important lesson is not the number of affected functions.

It is that a single undocumented boundary assumption propagated across unrelated features.

---

## Why Point Fixes Were Not Enough

A local patch could normalize values at each comparison:

```python
if str(proj.owner_user_id) != str(owner_user_id):
    ...
```

Similar conversions could be added before JSON serialization, export filtering, and Pydantic model construction.

That would repair the paths already discovered.

It would not make the system's contract explicit.

Every new code path would still be able to reproduce the same failure.

The problem therefore needed to be fixed at the shared boundary rather than at every symptom.

---

## The Fix

The application already treated identifiers as strings across routes, schemas, payloads, and tests.

The persistence model was changed to match that existing application contract:

```python
from sqlalchemy.types import Uuid

# PostgreSQL remains native UUID.
# Python-side ORM values are returned as strings.
UuidStr = Uuid(as_uuid=False)
```

UUID-backed model columns then use the shared alias:

```python
id: Mapped[str] = mapped_column(
    UuidStr,
    primary_key=True,
    server_default=text("gen_random_uuid()"),
)
```

The resulting contract is:

```text
Database representation  → native PostgreSQL UUID
ORM / application value  → str
API / JSON boundary       → str
```

This did not require changing the database's native UUID type, indexes, constraints, or foreign keys.

It changed only the Python-side representation returned by the ORM.

---

## Validation

The historical engineering record for the fix states:

- The shared alias was applied across 45 UUID column declarations.
- The real-PostgreSQL Tier 1 suite passed after the change.
- The recorded full result was **58 passed**.
- No database migration was required.

That is evidence recorded with the original fix on 2026-05-20.

It is not presented here as a new test run performed for this article.

More importantly, the validation target was broader than “does the edited line compile?”

The relevant paths included:

```text
Real PostgreSQL read
+
Owner comparison
+
Event payload serialization
+
Export filtering
+
Application schema conversion
```

---

## Three-Layer Validation

This incident reinforced a three-layer validation discipline.

### Layer 1 — Compile and Mock Tests

Verify local logic, syntax, branch behavior, and fast regression coverage.

```text
Does the code run under the assumptions encoded by the test?
```

### Layer 2 — Real Infrastructure

Use the actual database, container paths, drivers, migrations, and runtime configuration.

```text
Do real dependencies preserve the assumptions made by the application?
```

### Layer 3 — End-to-End Business Result

Run the complete user path and inspect the outcome.

```text
Can the owner open the project?
Does the export contain the expected records?
Can the payload cross the API boundary?
```

Each layer answers a different question.

Passing one layer does not imply that the next layer is correct.

---

## Engineering Rule

# Mocks must preserve boundary behavior, not only example values.

When a system crosses a database, SDK, queue, filesystem, or LLM-provider boundary, verify at least:

- Runtime type
- Nullability
- Serialization behavior
- Ordering
- Error shape
- Retry behavior
- Ownership and identity semantics

A useful review question is:

> **What does the real dependency return that the mock has simplified away?**

---

## Relationship to Case #01, #02, and #03

This case continues the same reliability series.

### Case #01 — Tool Calling Silent Failure

The response reported a tool-call finish state, but the streamed tool payload was missing.

```text
Reported state
≠
Actual tool-call data
```

### Case #02 — Hidden Context Overhead

The visible prompt was small, but the effective context was much larger.

```text
Visible prompt
≠
Effective context
```

### Case #03 — Context Budget Silent Failure

The budget module reported an invalid state, but execution could continue unless downstream control flow enforced it.

```text
Error state
≠
Actual control flow
```

### Case #04 — Real Database Type Boundary Failure

The identifier looked the same, but the mock and the real database returned different runtime types.

```text
Visible value
≠
Runtime type
```

Together, the four cases point to one recurring pattern:

> Reliability failures often hide in the gap between a declared abstraction and the behavior that actually crosses its boundary.

---

## Takeaway

Do not stop at:

> “Did the tests pass?”

Also ask:

> **“Did the tests reproduce the behavior of the real dependency?”**

A green test suite is evidence.

It is not the business result.

---

## Evidence and Limits

This case is based on the recorded implementation and decision history of the `NeverReset V1.0` project.

Verified in the repository:

- `backend/memory_engine/models.py` defines `UuidStr = Uuid(as_uuid=False)`.
- `backend/memory_engine/projects.py` contains the owner comparison shown above.
- `docs/decision_log.md`, decision #50, records the shared root cause, affected paths, 45 UUID declarations, and the historical 58-test validation result.

Scope limits:

- This article does not claim that every mock-based test has this weakness.
- It does not claim an authorization bypass or cross-user exposure.
- It does not claim that the historical suite was rerun for this publication.
- The simplified IDs in diagrams are illustrative, not production identifiers.

---

## Principle

> **Don't assume it works. Prove it against the real boundary.**

AgentInfra Lab  
LLM Reliability · Agent Engineering
