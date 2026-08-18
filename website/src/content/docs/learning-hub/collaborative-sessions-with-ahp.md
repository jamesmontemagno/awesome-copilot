---
title: 'Collaborative Sessions with Agent Host Protocol (AHP)'
description: 'Learn how the Agent Host Protocol lets multiple terminals share the same Copilot session, attach to remote environments, and collaborate in real time.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-18
estimatedReadingTime: '8 minutes'
tags:
  - ahp
  - collaboration
  - multi-session
  - copilot-cli
relatedArticles:
  - ./copilot-configuration-basics.md
  - ./agents-and-subagents.md
  - ./understanding-mcp-servers.md
prerequisites:
  - GitHub Copilot CLI installed (v1.0.80+)
  - Basic understanding of GitHub Copilot agents
---

The **Agent Host Protocol (AHP)** is a client-host architecture built into Copilot CLI that lets multiple terminals share the same agent session. Instead of every terminal running its own isolated Copilot process, AHP separates the **host** (where the agent actually runs) from one or more **clients** (terminals you use to interact with it).

> **Feature flag note:** AHP is currently gated on the `AHP_CLIENT` feature flag. Access is rolling out progressively. If `--ahp` is not yet available in your CLI version, it will appear as you update.

## Why AHP Matters

Without AHP, every terminal that runs `copilot` starts its own independent process with its own sessions. If you open a new terminal, you lose the context from your previous conversation. With AHP:

- **Persistent sessions** — agent sessions live on the host process, not in your terminal window. Close a terminal, reopen it, and attach to exactly where you left off.
- **Multiple simultaneous clients** — two terminals (or two developers) can watch the same session stream live. Every turn is visible to all attached clients.
- **Remote compute** — you can run the agent on a Codespace or a Mission Control cloud environment and interact with it from your local terminal.
- **Session hand-off** — share your session with a colleague by pointing them at the same host.

## Quick Start

### Start a local host and attach

Launch a local AHP host and immediately attach to it:

```bash
copilot --ahp
```

If an AHP daemon is already running in a directory that covers your current working directory, Copilot attaches to it automatically. Otherwise it starts a new host and attaches.

### Attach to an existing host

To attach a second terminal to a host that is already running:

```bash
copilot --ahp
```

Running `--ahp` from any directory covered by an already-running daemon attaches you to that daemon — no address needed. You can also specify a host explicitly:

```bash
copilot --ahp "wss://localhost:8765"
```

## Managing Hosts with `/ahp` Commands

Once inside a session, use `/ahp` slash commands to manage hosts:

| Command | What it does |
|---------|-------------|
| `/ahp start [port]` | Start a new AHP daemon serving the current directory |
| `/ahp stop <host>` | Stop a running daemon (asks for `--force` if you're in the session) |
| `/ahp restart <host>` | Restart an existing daemon on its current workspace |
| `/ahp connect <url>` | Add a host to your source list live, without restarting the CLI |
| `/ahp hosts` | List all hosts the CLI knows about |
| `/ahp use <host>` | Switch the active source to a different host |
| `/ahp status` | Show identity, health, and connection info for all connected hosts |
| `/ahp sessions` | List sessions on the current host |
| `/ahp attach <session>` | Attach to a specific session on the current host |
| `/ahp new` | Create a new session on the current host |
| `/ahp codespace <name>` | Forward a Codespace's `copilotd` port and add it to the source picker |
| `/ahp cloud <env-id>` | Add a Mission Control cloud environment to the source picker |

## Sessions Tab

When `--ahp` is active, the **Sessions tab** (accessible with `h`) shows sessions from every connected source:

- **LOCAL** — sessions in this CLI process
- **AHP** — sessions from a connected daemon
- **CS** — a forwarded Codespace session
- **CLOUD** — a Mission Control cloud environment

Sessions that are actively running sort to the top, and each row shows the current state: running, waiting for input, or idle. Press `Enter` to join a session, `n` to create a new one on the selected host, or close a row to dispose of the session on the host.

## Connecting to Remote Environments

### Codespaces

Forward a Codespace's agent and open it in your Sessions tab:

```bash
# Inside a copilot --ahp session
/ahp codespace my-codespace-name
```

The command accepts the display name you gave the Codespace, not just the auto-generated identifier. It uses `gh` to set up port forwarding automatically. The session appears in the Sessions tab marked `CS`. The tunnel closes when you exit the CLI or run `/ahp stop <codespace-name>`.

> **Prerequisite**: Your `gh` token must include the `codespace` scope. If it is missing, the command tells you the exact `gh auth refresh` command to run.

### Mission Control Cloud Environments

Connect to a Mission Control environment so the agent runs on cloud compute:

```bash
/ahp cloud <environment-id>
```

The environment appears in the Sessions tab marked `CLOUD`. Mission Control wakes the environment on connect; the CLI cannot start or stop it independently.

## Multi-client Presence

When two clients are attached to the same session:

- The session row in the Sessions tab shows `2 clients` (or more).
- Joining is announced in the timeline when a new client attaches.
- Clients that disconnect stop being counted on the next heartbeat.
- All attached clients see turns stream in real time — including turns started by another client.

**Steering a running turn from any client**: While a turn is streaming, any attached client can:
- Press `Enter` to send a new message that steers the running turn.
- Press `Ctrl+Q` to queue a message for the next turn.
- Press `Ctrl+C` to cancel the current turn.

## Security Considerations

### Connection tokens

When connecting to a remote host (Codespace or LAN host), protect the connection with a token:

```bash
copilot --ahp "wss://host:8765?tkn=my-secret-token"
```

Copilot redacts the token from all output — `/ahp status`, session lists, error messages, and timeline notices — so transcripts can be shared without leaking credentials. A host that rejects the connection with `401` tells you that a token is required.

### Directory scope

An AHP session runs in the directory you started the CLI from, when the host's workspace covers it. A daemon serving a parent directory will not silently put your agent in the wrong project — it respects your starting directory within the host's workspace.

## Automatic Host Discovery

When you launch with `--ahp`, Copilot scans your machine for running AHP daemons and adds them to the Sessions tab source picker automatically. You do not need to name each host.

To disable auto-discovery:

```bash
COPILOT_AHP_DISCOVER=0 copilot --ahp
```

## Multiple Hosts

You can connect to more than one host at a time. Pass a comma-separated list to `--ahp`:

```bash
copilot --ahp "wss://host1:8765,wss://host2:8766"
```

Or add hosts live from inside a session with `/ahp connect <url>`. Use `h` to switch between sources in the Sessions tab — the selected source determines which sessions `n` creates in and which session list is shown.

## Common Questions

**Q: Does AHP require any special setup?**

A: No — `copilot --ahp` starts or attaches to a host automatically. The daemon starts in the background when none is running, or attaches to an existing one.

**Q: Can I still use regular sessions alongside AHP sessions?**

A: Yes. Local sessions (created without `--ahp`) continue to work normally. The Sessions tab shows both local and host sessions side by side.

**Q: What happens to an AHP session if I close all clients?**

A: The session continues running on the host. When you reattach with `--ahp`, your session is still there with its full history.

**Q: Can two people collaborate on the same session across different machines?**

A: Yes, if both machines can reach the same host (for example, via a Codespace or a host with a connection token). Both clients see turns stream live and can steer the session.

**Q: Does `/clear` or `/new` work in AHP sessions?**

A: Yes — `/clear` and `/new` create a replacement session **on the host**, not locally. Every attached client switches to the new session.

## Further Reading

- [GitHub Copilot CLI Releases](https://github.com/github/copilot-cli/releases) — release notes for v1.0.80+ where AHP was introduced
- [Copilot Configuration Basics](../copilot-configuration-basics/) — startup flags and session settings
- [Agents and Subagents](../agents-and-subagents/) — orchestration patterns for parallel work

---
