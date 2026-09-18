# Personal OpenCode setup, packaged for one-command setup on any machine.

Fully templated config (no hardcoded paths), autonomous orchestrator (default agent auto-decomposes and dispatches), 34 registered subagents, 126 skills (116 user + 10 config), 14 MCP servers.

## Features (best of)

- Autonomous delegation mandate with built-in routing tables for 34 agents
- `<CONFIG_ROOT>` portable path convention
- Context-first workflow (mandatory context loading)
- Env-template install (secrets never committed)
- One-command install + uninstall + backup

## Requirements

bash 4+, optional npm (for plugins), optional git

## Quickstart

```
git clone <repo-url> opencode-dotfiles && cd opencode-dotfiles
./install.sh
# then: nano ~/.config/opencode/env.sh   (fill OPENCODE_BIN_DIRS)
opencode auth login
opencode doctor
```

## What you get

| Component | Count |
|-----------|-------|
| Primary agents | 2 |
| Subagents | 34 |
| Commands | 9 + openagents/ subdir |
| Context | 7 subdirs + navigation.md |
| Skills | 126 (116 user + 10 config) |
| Plugins | 3 entries (opencode-skills-collection@latest, oh-my-opencode-slim, ./plugins/claude-mem.js) |
| MCP servers | 14 |
| Model mappings | 1 preset (opencode-go) |
| Agent metadata | 1 file (agent-metadata.json) |
| TUI config | 1 file (tui.json) |
| Env loader | 1 file (tool/env/index.ts) |
| Skill manifest | 1 file (.oh-my-opencode-slim/skills-manifest.json) |

## Environment variables

| Variable | Required? | Description |
|----------|-----------|-------------|
| OPENCODE_BIN_DIRS | ✅ Always | Colon-separated extra bin dirs (e.g. nodejs bin + `~/.local/bin`) prepended to local MCP servers' PATH |
| VOLTRA_ROOT | Optional | Filesystem MCP root |
| ARDUINO_SKETCH_ROOT | Optional | Arduino MCP sketch root (typically `$VOLTRA_ROOT/firmware`) |
| GITHUB_PERSONAL_ACCESS_TOKEN | Optional | GitHub MCP auth (env alternative to `{file:}` header) |
| TELEGRAM_BOT_TOKEN / TELEGRAM_CHAT_ID / TELEGRAM_BOT_USERNAME / TELEGRAM_IDLE_TIMEOUT / TELEGRAM_CHECK_INTERVAL / TELEGRAM_ENABLED | Optional | Telegram bot plugin |
| GEMINI_API_KEY | Optional | Optional LLM/provider |
| MINIMAX_API_KEY | Optional | Optional LLM/provider |

## Layout

```
opencode-dotfiles/
├── install.sh                 # main installer (bash, POSIX-ish, no deps beyond bash+coreutils)
├── uninstall.sh               # safe uninstaller
├── README.md                  # curated "best of" + quickstart
├── GLOBAL-MANIFEST.md         # portability manifest (source machine inventory)
├── env.sh.template            # env template (installer generates env.sh from it)
├── .gitignore                 # bundle-level ignores
├── LICENSE                    # MIT text (year 2026, author "izz")
├── opencode.jsonc             # COPIED from ~/.config/opencode/opencode.jsonc
├── opencode.json              # COPIED
├── oh-my-opencode-slim.json   # COPIED
├── tui.json                   # COPIED
├── AGENTS.md                  # COPIED
├── package.json               # COPIED (this IS tracked in bundle — see .gitignore note)
├── package-lock.json          # pinned deps for `npm install` (plugin support)
├── agent/                     # COPIED (core/, subagents/)
├── agents/                    # COPIED (23 files)
├── command/                   # COPIED (+ openagents/ subdir)
├── config/                    # COPIED (agent-metadata.json etc.)
├── context/                   # COPIED (7 subdirs + navigation.md)
├── plugins/                   # COPIED (claude-mem.js etc.)
├── skills/                    # COPIED (10 real + 105 category pointers)
├── skill-libraries/           # COPIED
├── tool/                      # COPIED (env/index.ts etc.)
├── .oh-my-opencode-slim/      # COPIED (skills-manifest.json)
└── skills-user/               # COPY of ~/.agents/skills (all 116 skill dirs)
    └── .skill-lock.json       # COPY of ~/.agents/.skill-lock.json
```

## Uninstall

`./uninstall.sh`

## Portability notes

- `{env:}` templating only from process env (no .env auto-load → env.sh + bashrc hook)
- auth.json machine-specific; must re-generate via `opencode auth` per machine
- GitHub PAT required per-machine; place at `~/.config/opencode/.secrets/github-pat` or export `GITHUB_PERSONAL_ACCESS_TOKEN`
- claude-mem worker: start with `npx claude-mem start &`
- model role mappings via oh-my-opencode-slim preset (opencode-go); reinstall with `bunx oh-my-opencode-slim@latest install --no-tui --skills=yes --background-subagents=yes`

## Troubleshooting

- MCP servers not launching → `OPENCODE_BIN_DIRS` not set
- GitHub MCP 401 → missing PAT; place at `~/.config/opencode/.secrets/github-pat`
- claude-mem silent → start worker: `npx claude-mem start &`
- skills missing → re-run `./install.sh --force`

## Footer

Link to: [GLOBAL-MANIFEST.md](GLOBAL-MANIFEST.md)