# Pi Agent Loop Deep Dive

**Date:** 2026-06-11
**Files analyzed:**
- `/Users/nick/github/earendil-works/pi/packages/agent/src/agent-loop.ts` (742 lines)
- `/Users/nick/github/earendil-works/pi/packages/agent/src/types.ts` (418 lines)
- `/Users/nick/github/earendil-works/pi/packages/agent/src/agent.ts` (557 lines)

---

## Architecture Overview

Pi's agent loop has three layers:

1. **`agent-loop.ts`** — Pure turn-sequencing loop (742 lines)
   - `agentLoop()` — Start a new prompt
   - `agentLoopContinue()` — Continue from current context
   - `runLoop()` — Main loop logic
   - `streamAssistantResponse()` — Stream LLM response
   - `executeToolCalls()` — Execute tool calls (sequential/parallel)

2. **`types.ts`** — Type definitions (418 lines)
   - `AgentMessage` — Union of `Message | CustomAgentMessages[keyof CustomAgentMessages]`
   - `AgentContext` — System prompt, messages, tools
   - `AgentLoopConfig` — Callbacks and settings
   - `AgentEvent` — Event types
   - `AgentTool` — Tool definition
   - `AgentToolResult` — Tool result

3. **`agent.ts`** — Stateful wrapper (557 lines)
   - `Agent` class — State management, event listeners, queue management
   - `PendingMessageQueue` — Steering and follow-up message queues
   - `ActiveRun` — Run lifecycle management

---

## Event Emission Order

### Pi's Event Sequence

```
agent_start
turn_start
message_start (for each prompt)
message_end (for each prompt)
  [Inner loop iteration]
  turn_start
  message_start (assistant response start)
  message_update (streaming deltas)
  message_end (assistant response end)
  tool_execution_start (for each tool call)
  tool_execution_end (for each tool call)
  message_start (tool result)
  message_end (tool result)
  turn_end
  [Check shouldStopAfterTurn]
  [Check steering messages]
  [Check follow-up messages]
agent_end
```

### Ion's Event Sequence

```
agent_start
turn_start
user_message (for each prompt)
  [Inner loop iteration]
  turn_start
  message_update (streaming deltas)
  agent_message (assistant response end)
  tool_call_start (for each tool call)
  tool_call_end (for each tool call)
  message_start (tool result)
  message_end (tool result)
  turn_end
  [Check shouldStopAfterTurn]
  [Check steering messages]
  [Check follow-up messages]
agent_end
```

### Differences

1. **Prompt events:** Pi emits `message_start`/`message_end` for each prompt. Ion emits `user_message`.
2. **Assistant response:** Pi emits `message_start` → `message_update` → `message_end`. Ion emits `message_update` → `agent_message`.
3. **Tool calls:** Pi emits `tool_execution_start`/`tool_execution_end`. Ion emits `tool_call_start`/`tool_call_end`.

**Impact:** Low — events serve the same purpose, just different names.

---

## Message Accumulation

### Pi's Model

```typescript
// Partial message is updated in-place during streaming
let partialMessage: AssistantMessage | null = null;

for await (const event of response) {
  switch (event.type) {
    case "start":
      partialMessage = event.partial;
      context.messages.push(partialMessage);
      break;
    case "text_delta":
      partialMessage = event.partial;
      context.messages[context.messages.length - 1] = partialMessage;
      break;
    case "done":
      const finalMessage = await response.result();
      context.messages[context.messages.length - 1] = finalMessage;
      break;
  }
}
```

**Key insight:** Pi pushes the partial message to `context.messages` immediately on `start`, then updates it in-place during streaming. This means the message is always in the context, even during streaming.

### Ion's Model

```go
// Accumulate content during streaming
var acc llm.StreamAccumulator
partialMessage := session.AgentMessage{}

for {
  chunk, ok := stream.Next()
  if !ok {
    break
  }
  if chunk.Content != "" {
    partialMessage.Message += chunk.Content
    l.emit(session.MessageUpdate{...})
  }
  acc.Add(chunk)
}

// Create final message from accumulated response
resp := acc.Response()
message := AgentMessage{
  Role:  "assistant",
  Parts: respParts(resp),
  ...
}
```

**Key insight:** Ion accumulates content in a `StreamAccumulator`, then creates the final message after streaming completes. The message is not in `l.state.Messages` until after streaming.

### Differences

1. **Partial message in context:** Pi pushes partial message to context immediately. Ion waits until streaming completes.
2. **Message updates:** Pi updates the partial message in-place. Ion accumulates content and creates message at the end.
3. **Streaming events:** Pi emits `message_start` at the beginning, `message_update` during, `message_end` at the end. Ion emits `message_update` during, `agent_message` at the end.

**Impact:** Medium — Pi's model allows other code to see partial messages during streaming. Ion's model is simpler but doesn't expose partial messages until completion.

---

## Error Handling

### Pi's Model

```typescript
// Stream must not throw — errors encoded in response
const response = await streamFunction(config.model, llmContext, {
  ...config,
  apiKey: resolvedApiKey,
  signal,
});

// Errors are encoded in the response via stopReason
if (message.stopReason === "error" || message.stopReason === "aborted") {
  await emit({ type: "turn_end", message, toolResults: [] });
  await emit({ type: "agent_end", messages: newMessages });
  return;
}
```

**Key insight:** Pi's stream function must not throw. Errors are encoded in the response via `stopReason: "error"` and `errorMessage`. The loop checks `stopReason` and exits gracefully.

### Ion's Model

```go
// Stream errors are returned as errors
stream, err := l.config.StreamFn(ctx, req)
if err != nil {
  return AgentMessage{}, llm.Message{}, fmt.Errorf("stream: %w", err)
}

// Stream errors during iteration
if err := stream.Err(); err != nil {
  return AgentMessage{}, llm.Message{}, fmt.Errorf("stream: %w", err)
}
```

**Key insight:** Ion's stream function can throw errors. Errors are wrapped and propagated up the call stack.

### Differences

1. **Error encoding:** Pi encodes errors in the response. Ion returns errors directly.
2. **Error handling:** Pi checks `stopReason` after streaming. Ion checks `stream.Err()` after iteration.
3. **Graceful degradation:** Pi always emits `turn_end` and `agent_end` even on error. Ion returns early on error.

**Impact:** High — Pi's model ensures events are always emitted in order. Ion's model can skip events on error.

---

## Cancellation

### Pi's Model

```typescript
// AbortSignal passed through to all operations
const response = await streamFunction(config.model, llmContext, {
  ...config,
  apiKey: resolvedApiKey,
  signal,
});

// Check signal at multiple points
if (signal?.aborted) {
  return {
    kind: "immediate",
    result: createErrorToolResult("Operation aborted"),
    isError: true,
  };
}
```

**Key insight:** Pi uses `AbortSignal` throughout, checking at multiple points in the loop.

### Ion's Model

```go
// context.Context used for cancellation
if ctx.Err() != nil {
  l.emit(session.TurnEnd{Base: session.BaseNow()})
  return newMessages, ctx.Err()
}
```

**Key insight:** Ion uses `context.Context` for cancellation, checking at the start of each iteration.

### Differences

1. **Mechanism:** Pi uses `AbortSignal`. Ion uses `context.Context`.
2. **Check points:** Pi checks at multiple points (stream, tool execution, loop). Ion checks at the start of each iteration.
3. **Granularity:** Pi can cancel individual tool calls. Ion cancels the entire loop.

**Impact:** Medium — both mechanisms work, but Pi's is more granular.

---

## Tool Execution

### Pi's Model

```typescript
// Parallel by default, sequential if any tool has executionMode: "sequential"
const hasSequentialToolCall = toolCalls.some(
  (tc) => currentContext.tools?.find((t) => t.name === tc.name)?.executionMode === "sequential",
);
if (config.toolExecution === "sequential" || hasSequentialToolCall) {
  return executeToolCallsSequential(...);
}
return executeToolCallsParallel(...);
```

**Key insight:** Pi checks both the global `toolExecution` config and per-tool `executionMode`. If any tool is sequential, all tools execute sequentially.

### Ion's Model

```go
// Parallel by default, sequential if any tool has executionMode: "sequential"
func (l *AgentLoop) shouldExecuteSequentially(calls []AgentToolCall) bool {
  for _, call := range calls {
    tool, ok := l.findTool(call.Name)
    if !ok {
      return true // unknown tool: sequential for safety
    }
    mode := tool.ExecutionMode
    if mode == "" {
      mode = l.config.ToolExecutionMode
    }
    if mode == ToolExecutionSequential {
      return true
    }
  }
  return false
}
```

**Key insight:** Ion also checks both global and per-tool execution modes. Unknown tools default to sequential for safety.

### Differences

1. **Unknown tools:** Pi doesn't have explicit handling for unknown tools. Ion defaults to sequential for safety.
2. **Tool validation:** Pi validates tool arguments. Ion also validates tool arguments.
3. **Tool hooks:** Both have `beforeToolCall` and `afterToolCall` hooks.

**Impact:** Low — both implementations are very similar.

---

## Steering and Follow-up Messages

### Pi's Model

```typescript
// Get steering messages at start of iteration
let pendingMessages: AgentMessage[] = (await config.getSteeringMessages?.()) || [];

// Inner loop: process tool calls and steering messages
while (hasMoreToolCalls || pendingMessages.length > 0) {
  // Inject pending messages
  if (pendingMessages.length > 0) {
    for (const message of pendingMessages) {
      await emit({ type: "message_start", message });
      await emit({ type: "message_end", message });
      currentContext.messages.push(message);
      newMessages.push(message);
    }
    pendingMessages = [];
  }
  
  // ... process tool calls ...
  
  // Get steering messages for next iteration
  pendingMessages = (await config.getSteeringMessages?.()) || [];
}

// Agent would stop here. Check for follow-up messages.
const followUpMessages = (await config.getFollowUpMessages?.()) || [];
if (followUpMessages.length > 0) {
  pendingMessages = followUpMessages;
  continue;
}
```

**Key insight:** Pi checks steering messages at the start of each iteration, and follow-up messages when the agent would otherwise stop.

### Ion's Model

```go
// Get steering messages at start of iteration
if len(pendingMessages) == 0 {
  pendingMessages = l.getSteeringMessages()
}

// Inner loop: process tool calls and steering messages
for hasMoreToolCalls || len(pendingMessages) > 0 {
  // Inject pending messages
  if len(pendingMessages) > 0 {
    l.state.Messages = append(l.state.Messages, pendingMessages...)
    newMessages = append(newMessages, pendingMessages...)
    // ...
    pendingMessages = nil
  }
  
  // ... process tool calls ...
  
  // Get steering messages for next iteration
  pendingMessages = l.getSteeringMessages()
}

// Agent would stop here. Check for follow-up messages.
followUpMessages := l.getFollowUpMessages()
if len(followUpMessages) > 0 {
  pendingMessages = followUpMessages
  continue
}
```

**Key insight:** Ion has the same logic as Pi for steering and follow-up messages.

### Differences

**None** — both implementations are identical in behavior.

---

## Prepare Next Turn

### Pi's Model

```typescript
const nextTurnSnapshot = await config.prepareNextTurn?.(nextTurnContext);
if (nextTurnSnapshot) {
  currentContext = nextTurnSnapshot.context ?? currentContext;
  config = {
    ...config,
    model: nextTurnSnapshot.model ?? config.model,
    reasoning:
      nextTurnSnapshot.thinkingLevel === undefined
        ? config.reasoning
        : nextTurnSnapshot.thinkingLevel === "off"
          ? undefined
          : nextTurnSnapshot.thinkingLevel,
  };
}
```

**Key insight:** Pi's `prepareNextTurn` can update context, model, and thinking level.

### Ion's Model

```go
if l.config.PrepareNextTurn != nil {
  l.applyTurnUpdate(l.config.PrepareNextTurn(turnContext))
  turnContext.Context = l.buildContext()
}
```

**Key insight:** Ion's `prepareNextTurn` can also update context, model, and thinking level.

### Differences

**None** — both implementations are identical in behavior.

---

## Should Stop After Turn

### Pi's Model

```typescript
if (
  await config.shouldStopAfterTurn?.({
    message,
    toolResults,
    context: currentContext,
    newMessages,
  })
) {
  await emit({ type: "agent_end", messages: newMessages });
  return;
}
```

**Key insight:** Pi's `shouldStopAfterTurn` receives the message, tool results, context, and new messages.

### Ion's Model

```go
if l.config.ShouldStopAfterTurn != nil {
  if l.config.ShouldStopAfterTurn(turnContext) {
    return newMessages, nil
  }
}
```

**Key insight:** Ion's `shouldStopAfterTurn` receives the same context.

### Differences

**None** — both implementations are identical in behavior.

---

## Key Findings

### 1. Event Emission Order
- **Pi:** `message_start` → `message_update` → `message_end` for assistant messages
- **Ion:** `message_update` → `agent_message` for assistant messages
- **Impact:** Low — events serve the same purpose

### 2. Message Accumulation
- **Pi:** Pushes partial message to context immediately, updates in-place during streaming
- **Ion:** Accumulates content in `StreamAccumulator`, creates message after streaming
- **Impact:** Medium — Pi's model allows other code to see partial messages during streaming

### 3. Error Handling
- **Pi:** Stream must not throw, errors encoded in response via `stopReason: "error"`
- **Ion:** Stream can throw errors, wrapped and propagated
- **Impact:** High — Pi's model ensures events are always emitted in order

### 4. Cancellation
- **Pi:** Uses `AbortSignal`, checks at multiple points
- **Ion:** Uses `context.Context`, checks at start of each iteration
- **Impact:** Medium — both work, but Pi's is more granular

### 5. Tool Execution
- **Both:** Parallel by default, sequential if any tool has `executionMode: "sequential"`
- **Impact:** None — implementations are identical

### 6. Steering and Follow-up Messages
- **Both:** Identical behavior
- **Impact:** None — implementations are identical

### 7. Prepare Next Turn
- **Both:** Identical behavior
- **Impact:** None — implementations are identical

### 8. Should Stop After Turn
- **Both:** Identical behavior
- **Impact:** None — implementations are identical

---

## Recommendations

### High Priority
1. **Fix error handling** — Ensure events are always emitted in order, even on error
2. **Add message_start/message_end events** — Align with Pi's event sequence

### Medium Priority
3. **Improve cancellation granularity** — Check context at more points in the loop
4. **Consider partial message exposure** — Allow other code to see partial messages during streaming

### Low Priority
5. **Align event names** — Use `tool_execution_start`/`tool_execution_end` instead of `tool_call_start`/`tool_call_end`

---

## Conclusion

Pi's agent loop is well-designed and Ion's implementation is very similar. The main differences are in event emission order and error handling. Pi's model ensures events are always emitted in order, even on error. Ion's model can skip events on error. This is the most important difference to fix.
