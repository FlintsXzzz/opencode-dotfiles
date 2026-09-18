# OpenCode Global Config — Portability Manifest

> Auto-generated 2026-09-16. Documents every globally-installed category,
> its source location, env dependencies, and portability status.
> Source machine: WSL Ubuntu, opencode 1.18.31, `/home/arfi`.

---

## 1. Config Categories

| # | Category | Global Location | Count | Portable? |
|---|----------|----------------|-------|-----------|
| 1 | **Primary agents** | `~/.config/opencode/agent/core/` | 2 (`openagent.md`, `opencoder.md`) | ✅ Content-only `.md` |
| 2 | **Subagent agents** | `~/.config/opencode/agent/subagents/` | 5 subdirs (code, core, development, system-builder) | ✅ Content-only `.md` |
| 3 | **Tool-call agents** | `~/.config/opencode/agents/*.md` | 23 files (ai-security-auditor → ux-architect) | ✅ Content-only `.md` |
| 4 | **Commands** | `~/.config/opencode/command/` | 9 `.md` + `openagents/` subdir | ✅ Content-only `.md` |
| 5 | **Context** | `~/.config/opencode/context/` | 7 subdirs (core, development, navigation.md, openagents-repo, project, project-intelligence, ui) | ✅ Content-only `.md` |
| 6 | **Context root** | `~/.config/opencode/AGENTS.md` | 1 file (CodeGraph + claude-mem boilerplate) | ✅ Content-only |
| 7 | **Real skills (config)** | `~/.config/opencode/skills/` | 10 dirs (codemap, context7, deepwork, oh-my-opencode-slim, reflect, simplify, task-management, verification-planning, worktrees, clonedeps) | ✅ Content + scripts |
| 8 | **Category-pointer skills** | `~/.config/opencode/skills/` | 105 dirs (`*-category-pointer` managed by oh-my-opencode-slim) | ✅ `.md` stubs |
| 9 | **User skills** | `~/.agents/skills/` | 116 dirs (from 18 source repos tracked in `.skill-lock.json`) | ✅ Content + scripts |
| 10 | **Skill lock** | `~/.agents/.skill-lock.json` | 105 tracked skills, 18 source repos | ✅ JSON manifest |
| 11 | **Plugins** | `~/.config/opencode/plugins/` | 1 file: `claude-mem.js` (bundled w/ zod v4.5.4; hooks session.idle/compacting, `claude_mem_search` tool) | ⚠️ Requires `npx claude-mem start` worker |
| 12 | **Plugin entries** | `opencode.jsonc` → `"plugins"` | 3: `opencode-skills-collection@latest`, `oh-my-opencode-slim`, `./plugins/claude-mem.js` | ✅ Refs (npm + local) |
| 13 | **MCP servers** | `opencode.jsonc` → `"mcp"` | 14 + gh_grep (plugin-provided) | ⚠️ Requires env vars (OPENCODE_BIN_DIRS, VOLTRA_ROOT) |
| 14 | **Auth** | `~/.local/share/opencode/` | `auth.json` (600), `mcp-auth.json` (600) | 🔒 Machine-specific, must re-gen |
| 15 | **Secrets** | `~/.config/opencode/.secrets/` | `github-pat` (600) | 🔒 PAT, must provide per-install |
| 16 | **Model role mappings** | `~/.config/opencode/oh-my-opencode-slim.json` | 1 file (openai + opencode-go presets) | ✅ JSON |
| 17 | **Agent metadata** | `~/.config/opencode/config/agent-metadata.json` | 1 file (openagent, opencoder names) | ✅ JSON |
| 18 | **TUI config** | `~/.config/opencode/tui.json` | 1 file | ✅ JSON |
| 19 | **Env loader** | `~/.config/opencode/tool/env/index.ts` | 1 file (searchPaths, verbose, override config) | ✅ TypeScript |
| 20 | **Skill manifest** | `~/.config/opencode/.oh-my-opencode-slim/skills-manifest.json` | 1 file (managed skills list) | ✅ JSON |
| 21 | **Formatters** | Per-project by opencode design | N/A globally | ⚠️ Binaries must be on PATH (e.g. `OPENCODE_BIN_DIRS` includes `~/.local/lib/nodejs/bin`) |

---

## 2. MCP Servers (14 total)

| Server | Type | Enabled | Machine-Specific? |
|--------|------|---------|-------------------|
| context7 | remote | ✅ | No |
| supabase | remote | ✅ | No |
| github | remote | ✅ | ⚠️ Uses `{file:~/.config/opencode/.secrets/github-pat}` |
| sequential-thinking | local | ✅ | ⚠️ Requires `OPENCODE_BIN_DIRS` on PATH |
| vercel | remote | ✅ | No |
| playwright | local | ✅ | ⚠️ Requires `OPENCODE_BIN_DIRS` on PATH |
| filesystem | local | ✅ | ⚠️ Requires `{env:VOLTRA_ROOT}` — WSL Windows mount |
| sentry | remote | ✅ | No |
| codegraph | local | ✅ | ⚠️ Requires `OPENCODE_BIN_DIRS` on PATH |
| firecrawl | remote | ✅ | No |
| exa | remote | ✅ | No |
| semgrep | local | ❌ | ⚠️ Requires `OPENCODE_BIN_DIRS` on PATH (disabled) |
| arduino | local | ✅ | ⚠️ Requires `{env:ARDUINO_SKETCH_ROOT}` (usually `$VOLTRA_ROOT/firmware`) |
| memory | local | ✅ | ⚠️ Requires `OPENCODE_BIN_DIRS` on PATH |
| gh_grep | remote | ✅ | No — **provided by `opencode-skills-collection@latest` plugin**, not core config |

---

## 3. Environment Variables Required

The global config resolves `{env:VAR}` **from the process environment only** — opencode does NOT auto-load config-dir `.env` files. The portable pattern is a machine-local `env.sh` (sourced from `.bashrc` via a guarded `[ -f ... ] && . ...` line) that the installer generates from a template, so the shared repo never contains machine paths.

| Variable | Purpose | Required? |
|----------|---------|-----------|
| `OPENCODE_BIN_DIRS` | Colon-separated extra bin dirs (e.g. nodejs bin + `~/.local/bin`) prepended to local MCP servers' PATH | ✅ Always (local MCPs fail without it) |
| `VOLTRA_ROOT` | Filesystem MCP root | Only if filesystem MCP used |
| `ARDUINO_SKETCH_ROOT` | Arduino MCP sketch root (typically `$VOLTRA_ROOT/firmware`) | Only if arduino MCP used |
| `PATH` | Base system PATH (suffix after `OPENCODE_BIN_DIRS`) | ✅ Always (system-provided) |
| `GITHUB_PERSONAL_ACCESS_TOKEN` | GitHub MCP auth (env alternative to `{file:}` header) | Optional |
| `TELEGRAM_BOT_TOKEN` / `TELEGRAM_CHAT_ID` / `TELEGRAM_BOT_USERNAME` / `TELEGRAM_IDLE_TIMEOUT` / `TELEGRAM_CHECK_INTERVAL` / `TELEGRAM_ENABLED` | Telegram bot plugin | Optional |
| `GEMINI_API_KEY` | Optional LLM/provider | Optional |
| `MINIMAX_API_KEY` | Optional LLM/provider | Optional |

---

## 4. What Stays Project-Local (NOT in global config)

| Item | Location | Why Local |
|------|----------|-----------|
| VOLTRA opencode.json | `/mnt/c/Users/Arfi/Downloads/VOLTRA/opencode.json` | WSL project-specific formatters |
| VOLTRA AGENTS.md | `/mnt/c/Users/Arfi/Downloads/VOLTRA/AGENTS.md` | Project-specific rules |
| VOLTRA .agents/ | `/mnt/c/Users/Arfi/Downloads/VOLTRA/.agents/` | Project-specific rules |
| OpenAgentsControl .opencode/ | `~/OpenAgentsControl/.opencode/` | Upstream project config |
| Project opencode.json | Any `./opencode.json` per project | Per-project format/tools overrides |

---

## 5. What Breaks on a Fresh Machine

| Issue | Impact | Fix |
|-------|--------|-----|
| `opencode.jsonc` hardcoded `/home/arfi` paths | MCP servers fail to launch | Templated via `{env:OPENCODE_BIN_DIRS}`, `{env:VOLTRA_ROOT}`, `{env:ARDUINO_SKETCH_ROOT}` |
| `opencode.jsonc` hardcoded `/mnt/c/...` paths | Filesystem/Arduino MCP fail | `{env:VOLTRA_ROOT}` / `{env:ARDUINO_SKETCH_ROOT}` |
| `auth.json` (600 perms) | Cannot share across users/machines | Re-generate via `opencode auth` |
| `.secrets/github-pat` (600 perms) | GitHub MCP auth fails | Provide PAT via `.secrets/` dir |
| `node_modules/` not installed | Plugins fail | `npm install` in config dir |
| `npx claude-mem start` worker not running | claude-mem plugin silent fail (session hooks + search tool unavailable) | Start worker: `npx claude-mem start &` |
| oh-my-opencode-slim not installed | Agent/model role mappings missing; orchestrator model (big-pickle) unmapped | `bunx oh-my-opencode-slim@latest install --no-tui --skills=yes --background-subagents=yes` |
| Skills not synced (116 ~/.agents/skills) | 116 user skills unavailable on fresh machine | Clone from source repos listed in `.skill-lock.json`, or run skill re-fetch script |

---

## 6. Portability Notes

- **Template syntax**: opencode supports `{env:VAR}` for env substitution and `{file:path}` for file contents in `opencode.jsonc`.
- **`.env` loading**: opencode does **NOT** auto-load config-dir `.env` — env vars must come from the process environment (installed via `env.sh` sourced from `.bashrc`).
- **Skills ecosystem**: 116 user skills + 10 config skills + 105 category pointers; all content-only (`.md` + scripts), no compiled binaries.
- **claude-mem plugin**: Bundled JS with zod v4.5.4; hooks opencode session events; requires `npx claude-mem start` worker process.
- **oh-my-opencode-slim**: Manages agent/model role mappings and skill assignments; installed via `bunx oh-my-opencode-slim@latest install --no-tui --skills=yes --background-subagents=yes`.
- **opencode-skills-collection**: Plugin by FrancoStino; skill marketplace with 1595+ skills via SkillPointer.

---

## 7. Source Repos (for skill re-fetch)

Skills in `~/.agents/skills/` are tracked in `.skill-lock.json` with source URLs.
Key upstream repos (skill count):

| Repo | Skills | URL |
|------|--------|-----|
| jeffallan/claude-skills | 67 | https://github.com/jeffallan/claude-skills |
| obra/superpowers | 14 | https://github.com/obra/superpowers |
| subsy/ralph-tui | 5 | https://github.com/subsy/ralph-tui |
| anthropics/skills | 4 | https://github.com/anthropics/skills |
| playwright-mcp-skills | ~3 | https://github.com/nicholasoxford/playwright-mcp-skills |
| vercel-labs/vercel-skills | ~2 | https://github.com/vercel-labs/vercel-skills |
| Other repos | ~11 | Various |

---

*This manifest is part of the portable opencode-dotfiles bundle.*
*See `install.sh` for automated setup on a new machine.*