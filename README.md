# MCP Command Injection Research

Independent security research on OS command injection (CWE-78) vulnerabilities in
open-source Model Context Protocol (MCP) servers.

**Full writeup:** <MEDIUM_LINK>

## Summary

I scanned ~95 MCP server projects and identified **four independently validated
OS command injection vulnerabilities** across four repositories. Each was confirmed
with a working proof of concept and responsibly disclosed with a proposed fix.
All testing was performed only against local instances I controlled.

## Findings

| # | Project | Tool / parameter | Root cause | Disclosure |
|---|---------|------------------|-----------|------------|
| 1 | maoxiaoke/mcp-media-processor | image tools / output filename | `execSync` with interpolated input | GitHub issue + VulDB |
| 2 | ckvv/mcp-server-music | `play` / `name` | `exec(osascript -e '...')` quote breakout | Issue + Fix PR #2 (maintainer: unmaintained demo) |
| 3 | bifteki-crew/GodotPlay | `godot_launch` / godotPath, projectPath, scene | unquoted `exec()` | Fix PR #10 |
| 4 | DAFoam/dafoam_mcp_server | `flow_field` (x2), `log_file` | `subprocess.run(["bash","-c", ...])` | Fix PR #7 |

## The common anti-pattern

All four shared one root cause: an untrusted tool parameter concatenated into a
shell command string and executed through a shell. Shell metacharacters
(`;`, `|`, `&&`, `$()`) are then interpreted as commands.

## The fix pattern

1. **Avoid the shell** — use `execFile` / `spawn` (Node) or an argv list (Python)
   so values are passed as literal arguments.
2. **Validate input** — whitelist enumerated values or restrict to a safe character set.

## Fix pull requests

- GodotPlay: https://github.com/bifteki-crew/GodotPlay/pull/10
- DAFoam: https://github.com/DAFoam/dafoam_mcp_server/pull/7
- mcp-server-music: https://github.com/ckvv/mcp-server-music/pull/2

## Status

Findings disclosed; CVE IDs requested via VulDB (in moderation).

---

Author: Ahad Nawaz
