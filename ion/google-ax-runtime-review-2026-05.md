---
date: 2026-05-20
summary: Review of google/ax as a distributed agent runtime reference for Ion/Canto.
status: active
---

# Google AX Runtime Review

## Snapshot

- Repo: <https://github.com/google/ax>
- Reviewed commit: `8946475c236a7ce30e81b47393937ae1b1855eed`
- Date checked: 2026-05-20
- Verification run in local clone: `go test ./...`; `go vet ./...`

## What AX Is

AX is a distributed execution/runtime layer, not a model harness or coding-agent
TUI. Its README explicitly frames it as early active development with breaking
resumption protocol changes expected before stable release.

Core shape:

- Single controller coordinates executions, registry, and event log.
- Conversation log is client-visible/session history; execution log is internal
  replay/resumption state.
- Client catch-up is explicit with `last_seq`; branching is explicit via `fork`.
- AgentService is a remote-agent gRPC boundary; A2A, ADK, Colab, substrate, and
  skills are integration surfaces.
- Confirmation content is a durable pending checkpoint for approval-required
  tools.

## Useful Patterns For Ion/Canto

- Preserve the current Canto/Ion split where the runtime owns ordered execution
  events and Ion projects display state. AX reinforces a single-writer runtime
  model rather than TUI-owned runtime state.
- Consider a first-class distinction between provider-visible/user-visible
  history and private execution/recovery state. AX has `internal_only` messages
  that are kept in execution logs but hidden from conversation history, clients,
  and agents.
- Treat reconnect/resume as client catch-up over an ordered stream, not as turn
  rewind. AX `last_seq` is a clean model for future remote/server mode.
- Keep fork/branch as a log operation with an explicit source sequence or
  snapshot. This is Phase 2 material, not Phase 1.
- Approval-required actions can be represented as durable pending checkpoints:
  record the tool call plus confirmation question, resume by replaying the
  question until an answer arrives, and emit a tool result on denial to avoid a
  dangling provider tool call.
- Optional subagents should stay registry/tool-like at the runtime boundary.
  AX's planner exposes registered agents as function tools, but its subagent
  resumption and approvals are still roadmap items.

## Cautions

- Do not copy AX protocols directly yet. The repo says resumption and agent
  communication protocols are unstable.
- AX is infrastructure-oriented and Kubernetes/substrate-friendly. Ion's Phase 1
  should stay a local terminal agent; AX belongs in Pi+ reference material.
- Some implementation details are not mature enough as-is: SQLite conversation
  sequence assignment uses `MAX(seq)+1` outside an explicit transaction, remote
  agents cannot start nested executions yet, and subagent confirmation behavior
  is carried through context with a comment noting it may need durable config.

## Implications

- Add Google AX to the Canto framework reference map.
- For `tk-8z4a`, no immediate code change is justified from AX alone. The
  useful near-term lesson is to keep Ion's adapter thin and avoid creating a
  second execution log or hidden runtime owner in Ion.
- For Phase 2, AX is a stronger reference than generic background-agent notes
  for remote/server mode, branch/fork, resumable streams, durable approvals,
  and isolated subagent execution.
