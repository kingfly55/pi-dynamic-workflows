# Divergences from upstream

This is a personal fork of
[QuintinShaw/pi-dynamic-workflows](https://github.com/QuintinShaw/pi-dynamic-workflows).
It exists for one reason: **let workflow subagents use tools registered by
specific host extensions** (an MCP bridge, browser tools, …), which upstream
deliberately does not allow.

Upstream builds one shared, extension-free resource loader for all subagents
in a run (the #109 memory-leak mitigation), so subagents never see
host-extension-registered tools. That default is preserved here — with no
configuration, this fork behaves exactly like upstream. The only functional
divergence is an opt-in allowlist that loads named extensions back into
subagent sessions.

## Divergence 1: `subagentExtensions` allowlist

- **Setting:** `subagentExtensions` in `~/.pi/workflows/settings.json`
  (or the repo-local / per-project override layers — same precedence as all
  other settings). Also available as `subagentExtensions` on
  `WorkflowManagerOptions` / `WorkflowAgentOptions` for library embedders.
- **Behavior:** each entry matches against a host extension's install path
  (case-insensitive substring) or exactly against the install directory name.
  When non-empty, subagents load *only* the matching extensions; empty or
  omitted keeps the upstream no-host-extensions behavior.
- **Safety preserved:** the allowlisted extensions load once per run and are
  shared by all subagents (the #109 mitigation degrades only by the size of
  the allowlisted set), and the `workflow`/`workflow_control` tool denylist
  (#107) still applies on top — an allowlisted extension cannot reintroduce
  recursive orchestration.
- **User docs:** [README § Subagent host-extension allowlist](README.md#subagent-host-extension-allowlist).
- **Branch:** `feat/subagent-extension-allowlist`.

## Maintenance notes

- This fork tracks upstream `main`; the divergence is intentionally small
  (settings + manager + agent loader filter + docs) to keep rebasing cheap.
- `npm run build` is required before `test:unit` on a fresh checkout (the
  packaging tests assert against `dist/`; upstream has the same property).
