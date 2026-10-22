# Guardix

Guardix runs security audits on smart-contract repositories. Connect a repository, start and track an audit, and review findings from the command line or from a coding agent.

This repository publishes the Guardix CLI and the Guardix plugin for Claude Code and Cursor.

https://guardix.io

Contact: [guardix@dome.net](mailto:guardix@dome.net)

## Install the CLI

macOS and Linux:

```bash
curl -fsSL https://github.com/guardixdev/guardix/releases/latest/download/install.sh | sh
```

The installer picks the binary for your OS and architecture, checks the checksum, and installs to `/usr/local/bin` (or `~/.local/bin` when that directory needs no extra privileges).

Windows and pinned versions: download an archive from the [releases](https://github.com/guardixdev/guardix/releases) page and put `guardix` on your `PATH`.

## Example

`guardix version` prints the build you have installed. This is the output of the Linux build from the current latest release, v0.3.3:

```text
guardix version
guardix 0.3.3 (commit 73faffe2f, built 2026-07-15T06:49:27Z)
```

`guardix --help` lists the rest of the surface, including `init`, `status`, `audit`, `finding`, and `doctor`.

## Plugin

The same repository is the plugin marketplace.

Claude Code:

```text
/plugin marketplace add guardixdev/guardix
/plugin install guardix-cli@guardix
```

The plugin includes three skills:

- `guardix` reads audit findings for the repository you have open
- `running-audits` starts an audit and follows it until it finishes
- `cross-audit-triage` compares findings that come from more than one auditor

The Cursor plugin manifest is `.cursor-plugin/plugin.json`. The same skills are under `skills/`.
