# goatherdai

Local-first Go software stack for AI agents that do real work on your machine.

Everything here ships as source and plain binaries, keeps its state in files
you own (SQLite, git worktrees, containerd snapshots), and assumes no accounts,
no telemetry, and no hosted service.

## The herd

- **[yaah](https://github.com/goatherdai/yaah)** — a vendor-free agent harness.
  One Go binary orchestrates a team of AI coding agents: the orchestrator plans
  and judges, sub-agents research, implement, test, and review in parallel
  inside isolated worlds, and their work comes back as sealed proposals the
  orchestrator reviews before anything reaches your tree. Speaks any
  OpenAI-compatible API or the Anthropic Messages API, stores sessions and
  memory in local SQLite, and speaks MCP as both client and server.

- **[goatherd](https://github.com/goatherdai/goatherd)** — the layer
  underneath: a content-addressed, append-only execution trace for agents, plus
  the scope tree, sandboxed workspaces, and supervision rules to steer a run in
  real time. The runtime adds execution scopes, the merge gate,
  settlement/recovery, sandbox backends (Landlock, Seatbelt, winacl), tasks,
  and supervision. (Ships as `goatherd-kernel` and `goatherd-runtime`.)

- **[croft](https://github.com/goatherdai/croft)** — an isolated,
  snapshot-backed workspace on containerd's overlayfs snapshotter: provisioned
  from an image, worked in-band, and captured into states that outlive the
  sandbox — cheap checkpoints, exact O(1) rollback, and forks at layer-sharing
  cost. Linux-only, deliberately not a container runtime.

## How they stack

```
yaah           the harness you run — agents, tools, MCP, TUI/REPL/web
 └─ goatherd   scopes, dispatch, settlement, supervision, sandbox backends
     └─ croft  containerd overlayfs workspaces with capture/rollback/fork
```

Each layer is usable on its own and honest about what it does not do. yaah
runs fine without croft (worktree isolation by default); croft knows nothing
about agents; goatherd steers scopes without caring which harness created
them.

## Principles

- **Vendor-free.** The stdlib first; every direct dependency earns its place.
  No SDKs that hide the wire format, no agent-platform accounts.
- **Your machine.** State lives in local files — SQLite, git worktrees,
  overlayfs layers — never behind someone's API.
- **Isolation you can name.** Where the host can confine a dispatch, it's
  kernel-enforced (Landlock, Seatbelt, or winacl) with a writable root the
  process chose; where it can't, the docs say so.
- **Reviewable by default.** Agents propose, the orchestrator disposes — a
  rejected attempt costs one dispatch and leaves your tree untouched.
- **Permissive licenses.** MIT (yaah is MIT OR Apache-2.0).
