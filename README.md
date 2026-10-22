# Guardix

Guardix runs AI security audits on Solidity smart-contract repositories and reports each finding with its severity, location, and evidence.

This repository holds three parts that connect your terminal and your coding agent to [Guardix](https://guardix.io):

| Part | What it does | Where it lives |
|------|--------------|----------------|
| The `guardix` CLI | Signs in to Guardix, connects a repository, starts and follows audits, and reads findings. | Binaries on the [Releases](https://github.com/guardixdev/guardix/releases) page |
| The Claude Code and Cursor plugin | Installs the Guardix skills into your coding agent. | [`.claude-plugin/`](.claude-plugin/), [`.cursor-plugin/`](.cursor-plugin/), and [`cli/plugin/`](cli/plugin/) |
| The skills | Tell the agent when to call the CLI and which commands exist. | [`skills/`](skills/) |

Website: [guardix.io](https://guardix.io). Docs: [CLI and agent skills](https://guardix.io/docs/guides/cli-and-agent-skills/).

## Install the CLI

On macOS and Linux, run the installer:

```bash
curl -fsSL https://github.com/guardixdev/guardix/releases/latest/download/install.sh | sh
```

The installer picks the binary for your OS and architecture and checks it against `checksums.txt`. It installs `guardix` to `/usr/local/bin`. If that directory is not writable, it installs to `~/.local/bin`.

On Windows, download [`guardix_0.3.4_windows_amd64.zip`](https://github.com/guardixdev/guardix/releases/download/v0.3.4/guardix_0.3.4_windows_amd64.zip) or [`guardix_0.3.4_windows_arm64.zip`](https://github.com/guardixdev/guardix/releases/download/v0.3.4/guardix_0.3.4_windows_arm64.zip) from the [Releases](https://github.com/guardixdev/guardix/releases) page and put `guardix.exe` on your `PATH`.

This README describes v0.3.4, the latest release.

## Check the install

`guardix version` prints the build you have. This is the real output of the v0.3.4 Linux build:

```text
$ guardix version
guardix 0.3.4 (commit 7c04cbbd5, built 2026-10-22T08:36:31Z)
```

`guardix doctor` checks your config, your sign-in, and the connection to the Guardix API.

## Sign in and start an audit

Run these commands inside a git checkout of the repository you want to audit:

1. Sign in and connect the repository:

    ```bash
    guardix init
    ```

    The CLI prints a URL and a pairing code. Approve the pairing in your browser at guardix.io. When you have no connected repositories, it asks you to connect the GitHub repository in this checkout. Press Enter to connect it. For CI, see `guardix auth login --help` for API key sign-in.

2. See the free audits left and the price of one audit:

    ```bash
    guardix quota
    ```

3. Start an audit of the current branch and follow it until it finishes:

    ```bash
    guardix audit start --wait
    ```

4. Check the latest audit and how many findings are still open:

    ```bash
    guardix status
    ```

An audit takes 2 to 10 hours, depending on the size of the code. To follow a running audit or read one finding in full, see `guardix audit --help` and `guardix finding --help`. Add `--json` to any command to get structured output that a script or an agent can parse.

## CLI commands in v0.3.4

| Command | What it does |
|---------|--------------|
| `guardix agent` | Installs and inspects Guardix integrations for coding agents. |
| `guardix audit` | Starts, lists, follows, and cancels audits, and downloads the PDF report. |
| `guardix auth` | Signs in, signs out, and shows the sign-in status. |
| `guardix billing` | Pays for audits and manages invoices. |
| `guardix completion` | Prints a shell completion script. |
| `guardix config` | Shows, sets, and removes saved CLI settings. |
| `guardix doctor` | Checks the setup, the sign-in, and the connection to the API. |
| `guardix finding` | Lists findings, shows one finding, and records a review or a comment. |
| `guardix init` | Signs in, and connects this checkout when you have no repositories yet. |
| `guardix manifest` | Prints every command, flag, environment variable, and exit code as one JSON document. |
| `guardix quota` | Shows the free audits left and the price of one audit. |
| `guardix release-audit` | Reviews external audit reports against release source and deployments. |
| `guardix repo` | Lists, connects, and disconnects repositories. |
| `guardix status` | Shows the latest audit and the open-finding count for the current repository. |
| `guardix team` | Shows and selects the active team. |
| `guardix upgrade` | Installs the latest published release. |
| `guardix version` | Prints the version, the commit, and the build date. |

## Install the plugin in Claude Code

Run these two commands in Claude Code:

```text
/plugin marketplace add guardixdev/guardix
/plugin install guardix-cli@guardix
```

The plugin calls the `guardix` CLI, so install the CLI and sign in first.

`npx skills add guardixdev/guardix` installs the same three skills into your coding agent.

## Use the plugin in Cursor

The Cursor plugin manifest is [`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json). It uses the same skills, which are in [`skills/`](skills/).

## What the skills do

The plugin installs three skills. The `guardix` skill needs CLI 0.3.4.

- `guardix` reads the audit findings for the repository you have open.
- `running-audits` starts an audit and follows it until it finishes.
- `cross-audit-triage` compares findings from several auditors, such as Guardix, an audit firm's report, and Slither output. It runs on your machine over the JSON output of the CLI and checks each finding against the code on disk.

## Results on EVMBench

EVMBench is an open benchmark from OpenAI and Paradigm with 117 known high-severity bugs from 40 audits. In the September 2026 run, the Guardix audit pipeline detected 104 of the 117 bugs, or 88.9%. The method, the full table, and other tools' published results are on the [EVMBench results page](https://guardix.io/benchmarks/evmbench/).

## How Guardix handles your code

Guardix clones your code to run the audit and does not train models on your code. The [security page](https://guardix.io/security/) lists what Guardix reads, what it keeps, and how to get your data deleted.

## Contact

Email [guardix@dome.net](mailto:guardix@dome.net), or start an audit at [guardix.io](https://guardix.io).

## License

The contents of this repository are under the [MIT License](LICENSE).
