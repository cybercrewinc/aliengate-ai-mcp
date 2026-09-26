<div align="center">

# AlienGate AI MCP

### AI-Driven Offensive Security Automation Platform

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![MCP](https://img.shields.io/badge/MCP-Compatible-purple.svg)](#)
[![Security Tools](https://img.shields.io/badge/Security%20Tools-150%2B-brightgreen.svg)](#)
[![Maintained by](https://img.shields.io/badge/Maintained%20by-CyberCrew-red.svg)](https://cybercrew.co.jp)

**A Model Context Protocol (MCP) server that lets an AI agent plan and run real security assessments — reconnaissance, web testing, network scanning, binary analysis, cloud/container review and CTF tooling — across 150+ integrated tools, with full activity logging of every scan.**

</div>

---

## What is this?

**AlienGate AI MCP** connects an AI assistant (any MCP-compatible client such as Claude Desktop, Claude Code, or Cursor) to a large arsenal of security tooling through a single, well-defined protocol. Instead of copy-pasting commands, the AI agent calls typed MCP tools; the server executes the underlying utilities on your testing box, captures the output, and returns structured results back to the agent.

It is designed for **authorized security work** — penetration testing engagements, red-team exercises, bug-bounty research, CTF competitions, and security research on systems you own or are explicitly permitted to test.

The platform ships as **two cooperating programs**:

| Component | File | Role |
|-----------|------|------|
| **API Server** | `aliengate_server.py` | Runs on your testing host. Exposes 150+ tools over an HTTP API, executes commands, manages processes, caches results, and writes the activity log. |
| **MCP Client** | `aliengate_mcp.py` | Registers 150+ MCP tools and forwards each agent request to the API server. This is what your MCP-compatible AI client talks to. |

## Key features

- **150+ integrated security tools** — network & recon, web application testing, authentication/password attacks, binary analysis & reverse engineering, cloud & container security, CTF & forensics, and OSINT.
- **151 typed MCP tools / 156 API endpoints** — every capability is a first-class, parameterized tool the AI agent can call directly.
- **Intelligent decision engine** — target analysis, technology detection, automatic tool selection, parameter optimization, and attack-chain construction.
- **Workflow automation** — ready-made bug-bounty reconnaissance, vulnerability-hunting, business-logic and OSINT workflows, plus CTF auto-solving helpers.
- **Resilient execution** — automatic retries with backoff, reduced-scope retries, alternative-tool fallback, and graceful degradation on failure.
- **Process management** — async execution, a scalable worker pool, live process dashboard, and pause/resume/terminate controls.
- **Full scan logging** — every command and scan is recorded (see below).

## Full scan logging

Auditability is built in. **Every scan and command executed through the platform is logged**, because all execution funnels through a single instrumented path.

Two log files are written next to the server:

- **`aliengate.log`** — general application log (startup, tool status, recovery actions, errors).
- **`aliengate_scans.log`** — a structured **JSON-lines** audit trail. One line per event:
  - `tool_invocation` — a tool run was requested (tool name, command, parameters).
  - `scan_start` — a command began (scan id, full command).
  - `scan_cache_hit` — a cached result was served.
  - `scan_end` — a command finished (success, return code, duration, stdout/stderr size, output preview).

Example line from `aliengate_scans.log`:

```json
{"timestamp": "2026-09-26T20:15:04.812345", "event": "scan_end", "scan_id": "scan_1790000000000_4821", "command": "nmap -sV 10.0.0.5", "success": true, "return_code": 0, "duration": 12.44, "stdout_bytes": 3820, "stderr_bytes": 0, "stdout_preview": "Starting Nmap ..."}
```

Change the scan-log path with the `ALIENGATE_SCAN_LOG` environment variable. Logs are git-ignored by default so evidence stays on the testing host.

## Installation

```bash
# 1. Clone
git clone https://github.com/cybercrewinc/aliengate-ai-mcp.git
cd aliengate-ai-mcp

# 2. Create a virtual environment
python3 -m venv aliengate_env
source aliengate_env/bin/activate      # Windows: aliengate_env\Scripts\activate

# 3. Install Python dependencies
python3 -m pip install -r requirements.txt

# 4. Start the API server
python3 aliengate_server.py
```

The server listens on `http://127.0.0.1:8888` by default. Override with environment variables:

```bash
ALIENGATE_HOST=0.0.0.0 ALIENGATE_PORT=8888 python3 aliengate_server.py
```

> **External tools:** the Python packages are only the framework. The 150+ security tools (nmap, gobuster, ffuf, nuclei, sqlmap, amass, subfinder, hydra, john, ghidra, radare2, trivy, prowler, volatility3, and many more) must be installed separately. A Kali/Parrot testing box has most of them out of the box; missing tools simply return a clear "not installed" error.

## Connecting an MCP client

Point your MCP-compatible AI client at the client script. Example configuration (`aliengate-ai-mcp.json`):

```json
{
  "mcpServers": {
    "aliengate-ai": {
      "command": "python3",
      "args": [
        "/path/to/aliengate_mcp.py",
        "--server",
        "http://127.0.0.1:8888"
      ],
      "description": "AlienGate AI MCP - AI-driven offensive security automation platform.",
      "timeout": 300,
      "alwaysAllow": []
    }
  }
}
```

Keep `alwaysAllow` empty to require manual approval before the agent runs anything autonomously.

## Health check

```bash
curl http://127.0.0.1:8888/health
```

Returns server status, version, and tool availability.

## Authorized use only

This platform executes real offensive security tooling. Use it **only** against systems you own or have **explicit, written authorization** to test. You are responsible for complying with all applicable laws, contracts, and rules of engagement. The maintainers accept no liability for misuse. See `LICENSE` (MIT) for warranty and liability terms.

## License

Released under the **MIT License**. See [`LICENSE`](LICENSE).

---

<div align="center">

**AlienGate AI MCP** — maintained by **CyberCrew Inc.**

</div>
