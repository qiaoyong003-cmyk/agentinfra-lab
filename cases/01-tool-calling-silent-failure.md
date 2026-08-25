# 01 · Tool Calling Silent Failure

## Problem

The API returned successfully, but the Agent could no longer use tools.

Observed state:

- HTTP 200
- SDK no exception
- `finish_reason = tool_calls`
- actual `tool_calls = 0`

## Test Matrix

<img src="../assets/tool-calling-2x2-matrix.png" width="520" alt="Tool Calling 2x2 Test Matrix">

**Key observation:**  
Only the OpenAI Compatible + Streaming path failed.

The upstream response declared:

`finish_reason = tool_calls`

but the streaming response contained:

`delta.tool_calls = 0`
