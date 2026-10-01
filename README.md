# codex-auth

Lightweight yet full‑featured – a Bash tool that pools credentials, displays usage, enables interactive switching, and auto‑syncs, making Codex CLI multi‑account management effortless.

![codex-auth account list and interactive account picker](assets/codex-auth-preview.svg)

## Features

- Automatically syncs the current `~/.codex/auth.json` to the credential pool
- Displays usage and reset times for ChatGPT accounts
- Provides an interactive account picker
- Removes inactive accounts from the credential pool with confirmation
- Removes `installation_id` when switching to a different account
- Refreshes stale pooled ChatGPT credentials through Codex
- Adds newly authenticated accounts to the pool automatically
- Leaves Codex sessions and history untouched

## Requirements

- Bash
- `jq`
- `curl`
- [Codex CLI](https://github.com/openai/codex) (required when adding or refreshing an account)
- Python 3 (used to parse some timestamps and account details)

## Installation

Run the script directly:

```bash
chmod +x codex-auth.sh
./codex-auth.sh
```

Or install it in your local binary directory:

```bash
mkdir -p ~/.local/bin
install -m 755 codex-auth.sh ~/.local/bin/codex-auth
```

Make sure `~/.local/bin` is included in your `PATH`.

## Usage

```bash
# List accounts and usage
codex-auth

# Log in and save a new account
codex-auth login

# Switch accounts interactively
codex-auth switch

# Remove an inactive account from the pool interactively
codex-auth remove

# Refresh stale ChatGPT credentials in the pool
codex-auth refresh
```

In the account picker, use `↑` / `↓` to move, `Enter` to confirm, and `q` to quit.
For removal, confirm with `y`. Switch away from the active account before removing it, or the next run will add it back from `auth.json`.
`refresh` checks every pooled ChatGPT credential and runs an isolated, ephemeral Codex command only for sessions older than about eight days; API-key entries are skipped.

Default file locations:

| File | Path |
| --- | --- |
| Current credentials | `~/.codex/auth.json` |
| Credential pool | `~/.codex/auth-poll.json` |
| Codex configuration | `~/.codex/config.toml` |
| Installation identifier | `~/.codex/installation_id` |

Override the credential, pool, and configuration paths with the `CURRENT_AUTH_FILE`, `AUTH_POOL_FILE`, and `CONFIG_TOML` environment variables, respectively. The installation identifier path is derived from the directory containing `CURRENT_AUTH_FILE`.

The script may update `config.toml` to set `cli_auth_credentials_store = "file"`, so Codex uses `auth.json` for credential storage. Every command also ensures `daemon_auto_start = false` under `[features]`, adding the table or setting if missing, so the shared daemon does not interfere with using different accounts in separate sessions.

> [!NOTE]
> Setting `daemon_auto_start = false` does not stop a running daemon. Reboot to stop it, or finish any sessions using it and run:
>
> ```bash
> codex app-server daemon stop
> ```
>
> Subsequent CLI launches should leave it stopped unless [configuration overrides](https://learn.chatgpt.com/docs/config-file/config-basic#configuration-precedence) re-enable auto-start. Disable any separately configured startup service too.

Proxy settings are inherited from `HTTP_PROXY` and `HTTPS_PROXY`. If either is unset, the script also accepts its lowercase equivalent (`http_proxy` or `https_proxy`) and exports the uppercase value to child commands.

> [!WARNING]
> The credential pool contains access tokens or API keys. Do not share it or commit it to version control. The script sets its permissions to `600`. Running `codex-auth login` backs up the current credentials before starting a new Codex login flow.

## License

[MIT](LICENSE)
