---
title: 'Remote Sessions with Agent Host Protocol (AHP)'
description: 'Learn how to share and attach to Copilot CLI sessions across terminals, Codespaces, and Mission Control environments using the Agent Host Protocol.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-13
estimatedReadingTime: '8 minutes'
tags:
  - copilot-cli
  - remote-sessions
  - ahp
  - codespaces
relatedArticles:
  - ./agents-and-subagents.md
  - ./using-copilot-coding-agent.md
  - ./copilot-configuration-basics.md
prerequisites:
  - GitHub Copilot CLI installed (v1.0.80+)
  - Basic familiarity with Copilot CLI sessions
---

The **Agent Host Protocol (AHP)** lets you attach one or more Copilot CLI terminals to a shared session host — a `copilotd` daemon that holds the session state. This means multiple terminals can watch the same session stream live, a session running in a Codespace can be joined from your laptop, and a Mission Control cloud environment can be used as a persistent compute host for your CLI sessions.

This article explains what AHP is, when to use it, and how to get started.

## What Is AHP?

In the default Copilot CLI workflow, each terminal is its own self-contained session. AHP flips this model: instead of the session living inside the CLI process, the session lives on a **host daemon** (`copilotd`), and one or more CLI terminals **attach** to that host. The CLI becomes a display and input surface; the host holds the real session state.

```
Terminal A (attach)  ─┐
Terminal B (attach)  ─┤─→  copilotd daemon  →  Session & turns
Terminal C (attach)  ─┘
```

This enables several new workflows:

- **Multi-terminal collaboration** — two developers can watch the same session in real time
- **Cross-machine sessions** — start a session on your laptop, continue on another machine
- **Codespace integration** — run Copilot in a Codespace, attach from your local terminal
- **Mission Control cloud sessions** — run compute in Mission Control, interact locally

> **Note**: AHP and the `--ahp` flag are gated on the `AHP_CLIENT` feature flag in v1.0.80. If you don't see these commands, you may need to wait for a future stable release.

## Getting Started

### Attach to a Local Host

When you run `copilot --ahp`, the CLI looks for a `copilotd` daemon already running on your machine. If one is found, it attaches to it. If none is running, it starts a new daemon for the current directory and attaches:

```bash
copilot --ahp
```

To see which daemons are running locally and in your Sessions tab, they are discovered automatically. Turn off auto-discovery with:

```bash
COPILOT_AHP_DISCOVER=0 copilot --ahp <host-url>
```

### Work with Sessions on the Host

Once attached, the **Sessions tab** (`h` to switch) shows all sessions on the host, including ones started by other attached CLIs. You can:

- **Join** an existing session with `enter`
- **Create** a new session on the host with `n`
- **Close** a session with the close action (which closes it for all attached CLIs)

Each session row shows whether it is running, waiting on input, or idle. Busy sessions sort to the top.

### Manage Daemons with `/ahp`

From inside any `--ahp` session, the `/ahp` command family manages daemon lifecycle:

| Command | Description |
|---------|-------------|
| `/ahp start [port]` | Start a new daemon serving the current directory |
| `/ahp stop <host>` | Stop a daemon (asks for `--force` if you are currently in it) |
| `/ahp restart <host>` | Restart a daemon on the same workspace |
| `/ahp connect <url>` | Add a remote host live |
| `/ahp hosts` | List all known hosts and their health |
| `/ahp use <host>` | Switch the Sessions tab to show a different host |
| `/ahp status` | Show identity, health, and connection details for the active host |

## Attaching to a Codespace

If your project runs in a GitHub Codespace, you can forward the Codespace's `copilotd` port to your local machine and attach directly:

```bash
copilot --ahp
# then inside the session:
/ahp codespace my-codespace-name
```

The Codespace is connected via `gh` and appears in the Sessions tab, marked **CS**. You can switch to it with `h` and create or join sessions on it just like a local daemon.

When you exit or run `/ahp stop my-codespace-name`, the tunnel closes. If the `codespace` scope is missing from your `gh` token, the CLI tells you the exact `gh auth refresh` command to run.

> **Tip**: You can use the Codespace's display name (the friendly name you gave it) rather than the auto-generated name `gh` uses internally.

## Attaching to Mission Control Cloud Environments

Mission Control environments can be used as remote compute hosts. To connect:

```bash
/ahp cloud <environment-id>
```

The environment appears in the Sessions tab, marked **CLOUD**. Mission Control wakes the environment on connect. Unlike local daemons, the CLI cannot start or stop Mission Control environments — it only connects to ones that already exist.

Combine this with `--cloud` to provision and attach automatically:

```bash
copilot --cloud
```

The provisioned environment appears in the Sessions tab, and you can switch back to it later even after the original `--cloud` session ends.

## Attaching to Multiple Hosts

`--ahp` accepts a comma-separated list of host URLs, and `/ahp connect <url>` adds a new host live:

```bash
copilot --ahp "ws://host1:8765,ws://host2:8765"
```

Each host gets its own section in the Sessions tab. Use `h` to switch between sources. The source in force is shown as a highlighted chip; sessions from other hosts are listed separately.

## Security and Tokens

When connecting to a remote host that uses a connection token (e.g., a LAN host or Codespace), include the token in the URL:

```bash
copilot --ahp "wss://host:8765?tkn=…"
```

The CLI redacts the token from all output (`/ahp status`, logs, timeline notices), so a transcript can be shared safely. A `401` response from the host tells you the connection token is required.

## Sharing Sessions

In `--ahp` mode, a session shared with another CLI shows a presence indicator:

- In the Sessions tab, a shared session row shows `2 clients` (or more) when someone else is attached
- `/ahp status` reports the same count
- Presence is announced on attach and cleared when a client disconnects

Every attached terminal sees prompts, turns, and tool results stream live. Typing while a turn is streaming works the same as locally: `Enter` steers a prompt into the running turn, `Ctrl+Q` queues it for next, and `Ctrl+C` takes it back.

## Common Questions

**Can a session survive my terminal closing?**

Yes. The session lives on the daemon, not in your terminal. Close your terminal and re-attach with `copilot --ahp` — your session is still running.

**Can I use `/clear` or `/new` in an AHP session?**

Yes. `/clear` and `/new` replace the session **on the host**, so every attached terminal sees the change.

**Does AHP work with hooks?**

Hook state and skills are maintained by the host session. If you attach to a session that was started on a different machine, local hooks that reference machine-specific paths may not work as expected.

**What if the host stops responding?**

The Sessions tab and timeline both announce when a host goes offline. You can reconnect with `/ahp connect` or start a fresh local session as a fallback.

## Further Reading

- **Copilot CLI Changelog**: [v1.0.80 release notes](https://github.com/github/copilot-cli/releases/tag/v1.0.80-0)
- **Related**: [Agents and Subagents](../agents-and-subagents/) — orchestrating multi-agent workflows
- **Related**: [Copilot Configuration Basics](../copilot-configuration-basics/) — startup flags and CLI settings
- **Related**: [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — autonomous coding on GitHub

---
