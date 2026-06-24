# claude-code — design-mcp

## [claude-design-mcp-asset-staging]
created: 2026-06-24
tags: claude-code, mcp, design, assets, subagents
context: Importing a claude.ai/design project via the claude_design MCP (DesignSync) to reimplement a design in code.
finding: (1) Only the main session can call the claude_design MCP; spawned subagents cannot — stage all design assets in the orchestrator BEFORE dispatching builders. (2) DesignSync get_file returns small/medium files INLINE (into context) but PERSISTS large outputs to a tool-result file on disk — for binaries (fonts/images) decode base64 from the persisted file; never hand-transcribe base64 from context (it corrupts/truncates). (3) If a binary won't stage losslessly (no persisted file), degrade to a CDN/system-font fallback with a documented drop-in path rather than ship a corrupt asset.
recommendation: Orchestrator stages HTML reference + CSS tokens + images (persisted-file decode) into the repo first, then hands on-disk refs to subagents. Treat fonts/large binaries as optional fidelity with a fallback.
