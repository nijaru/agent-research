# Codex Agent Loop Analysis

**Date:** 2026-06-11
**Files analyzed:**
- `codex-rs/core/src/session/turn.rs` (2207 lines)
- `codex-rs/core/src/tools/parallel.rs` (380 lines)
- `codex-rs/core/src/agent/control.rs` (1302 lines)
- `codex-rs/core/src/thread_manager.rs` (1544 lines)

---

## Architecture Overview

Codex is a very complex system with multiple layers:

1. **Thread Manager** (`thread_manager.rs`) — Thread lifecycle, session management
2. **Agent Control** (`agent/control.rs`) — Agent spawning, fork management
3. **Session** (`session/turn.rs`) — Turn execution, streaming, tool calls
4. **Tools** (`tools/parallel.rs`) — Parallel tool execution with cancellation

## Key Differences from Pi/Ion

### 1. Thread-Based Architecture
Codex uses a thread-based architecture where each conversation is a thread. Threads can be forked, resumed, and managed independently. This is more complex than Pi/Ion's single-agent model.

### 2. Streaming Model
Codex uses a streaming model with `ResponseEvent` types:
- `Created` — Response created
- `OutputItemDone` — Output item completed
- `OutputItemAdded` — Output item added
- `ContentDelta` — Content delta
- `ReasoningContentDelta` — Reasoning content delta

This is similar to Pi's `AssistantMessageEvent` but more granular.

### 3. Tool Execution
Codex uses a `ToolCallRuntime` for parallel tool execution:
- Supports parallel execution with `supports_parallel` flag
- Uses `tokio::select!` for cancellation
- Has `AbortOnDropHandle` for cleanup
- Uses `parallel_execution` lock for sequential tools

This is similar to Pi/Ion's approach but more sophisticated.

### 4. Context Management
Codex has sophisticated context management:
- Auto-compaction when token limits are reached
- Pre-sampling compaction
- Context window management
- Token counting and estimation

This is more advanced than Pi/Ion's simple context management.

### 5. Hook System
Codex has a rich hook system:
- `run_pending_session_start_hooks` — Session start hooks
- `run_hooks_and_record_inputs` — Input hooks
- `run_turn_stop_hooks` — Turn stop hooks
- `run_legacy_after_agent_hook` — Legacy after-agent hooks

This is similar to Pi's hooks but more granular.

### 6. Error Handling
Codex has sophisticated error handling:
- Retryable errors with backoff
- Transport fallback (WebSocket to HTTPS)
- Context window exceeded handling
- Usage limit reached handling

This is more advanced than Pi/Ion's error handling.

### 7. Cancellation
Codex uses `CancellationToken` from `tokio_util`:
- Supports cancellation at multiple points
- Has `AbortOnDropHandle` for cleanup
- Uses `tokio::select!` for cancellation

This is similar to Pi's `AbortSignal` but more sophisticated.

---

## Key Insights

### 1. Thread-Based Architecture
Codex's thread-based architecture allows for:
- Forking conversations
- Resuming conversations
- Managing multiple agents
- Sharing context between threads

This is more complex than Pi/Ion's single-agent model but allows for more sophisticated workflows.

### 2. Streaming Model
Codex's streaming model is more granular than Pi/Ion's:
- Supports multiple output types (messages, reasoning, tool calls)
- Has separate events for different content types
- Supports plan mode and collaboration modes

This is more flexible than Pi/Ion's simpler model.

### 3. Tool Execution
Codex's tool execution is more sophisticated than Pi/Ion's:
- Supports parallel execution with cancellation
- Has `AbortOnDropHandle` for cleanup
- Uses `parallel_execution` lock for sequential tools
- Has `ToolCallRuntime` for managing tool calls

This is more robust than Pi/Ion's simpler model.

### 4. Context Management
Codex's context management is more advanced than Pi/Ion's:
- Auto-compaction when token limits are reached
- Pre-sampling compaction
- Context window management
- Token counting and estimation

This is more sophisticated than Pi/Ion's simple context management.

### 5. Error Handling
Codex's error handling is more sophisticated than Pi/Ion's:
- Retryable errors with backoff
- Transport fallback (WebSocket to HTTPS)
- Context window exceeded handling
- Usage limit reached handling

This is more robust than Pi/Ion's simpler error handling.

---

## Comparison with Pi/Ion

| Feature | Pi | Ion | Codex |
|---|---|---|---|
| Architecture | Single agent | Single agent | Thread-based |
| Streaming | AssistantMessageEvent | StreamAccumulator | ResponseEvent |
| Tool execution | Parallel/Sequential | Parallel/Sequential | Parallel/Sequential |
| Context management | Simple | Simple | Sophisticated |
| Hooks | beforeToolCall/afterToolCall | BeforeToolCall/AfterToolCall | Multiple hooks |
| Error handling | StopReason | Error return | Retryable errors |
| Cancellation | AbortSignal | context.Context | CancellationToken |

---

## Recommendations for Ion

### High Priority
1. **Add transport fallback** — Codex falls back from WebSocket to HTTPS on errors
2. **Add pre-sampling compaction** — Codex compacts before sampling to avoid token limits
3. **Add usage limit handling** — Codex handles usage limits gracefully

### Medium Priority
4. **Add thread-based architecture** — Codex's thread-based architecture allows for more sophisticated workflows
5. **Add plan mode** — Codex supports plan mode for collaboration
6. **Add multiple hooks** — Codex has more granular hooks than Pi/Ion

### Low Priority
7. **Add collaboration modes** — Codex supports collaboration modes
8. **Add extension system** — Codex has an extension system for tools

---

## Conclusion

Codex is a very complex system with sophisticated architecture. The main differences from Pi/Ion are:
1. Thread-based architecture
2. Sophisticated context management
3. Advanced error handling
4. Rich hook system

Ion should focus on adding:
1. Transport fallback
2. Pre-sampling compaction
3. Usage limit handling

These are the most important features that Codex has that Pi/Ion don't.
