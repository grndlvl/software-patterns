# Draft: Oh My OpenCode Setup

## Requirements (confirmed)
- Install and configure `oh-my-opencode` following the upstream installation guide.
- Always invoke OpenCode via `opencode-personal` (wrapper/alias that loads different credentials) instead of `opencode`.
- Provider enablement: OpenAI/ChatGPT Plus = YES.
- Auth status: you said you are already logged in.

## Technical Decisions
- Use the non-interactive installer (`bunx oh-my-opencode install --no-tui`) so it can run in automation and won\'t require TUI.
- Treat `opencode-personal` as the canonical entrypoint, which typically works by setting:
  - `XDG_CONFIG_HOME=$HOME/.config/opencode-personal`
  - `XDG_DATA_HOME=$HOME/.local/share/opencode-personal`
  - and then executing `opencode`.

## Research Findings
- The installer writes OpenCode plugin config at: `~/.config/opencode-personal/opencode/opencode.json`
- The installer writes oh-my-opencode config at: `~/.config/opencode-personal/opencode/oh-my-opencode.json`
- Attempting a direct provider login via `opencode auth login openai` can fail with "fetch() URL is invalid" (suggests the CLI expects a URL argument or interactive provider selection, not the literal string `openai`).

## Scope Boundaries
- INCLUDE: plugin install, config file verification, provider authentication verification, and day-to-day usage guidance using `opencode-personal`.
- EXCLUDE: changing your shell dotfiles/aliases unless explicitly requested.

## Open Questions
- Do you also want Claude enabled (it may already be configured in your environment), or should the plan be OpenAI-only?
- What does `opencode-personal auth list` show right now (to confirm the existing login is usable)?
