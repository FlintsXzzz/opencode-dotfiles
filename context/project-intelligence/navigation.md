<!-- Context: project-intelligence/nav | Priority: high | Version: 1.2 | Updated: 2026-09-12 -->

# Project Intelligence

> Start here for quick project understanding. These files bridge business and technical domains.

## Structure

```
<CONFIG_ROOT>/.config/opencode/context/project-intelligence/
├── navigation.md              # This file - quick overview
├── technical-domain.md        # Stack, architecture, technical decisions
├── technical-patterns.md      # Advanced patterns (realtime, WS, RLS, build)
├── business-domain.md         # Business context and problem statement
├── business-tech-bridge.md    # How business needs map to solutions
├── decisions-log.md           # Major decisions with rationale
└── living-notes.md            # Active issues, debt, open questions
```

## Quick Routes

| What You Need | File | Description |
|---------------|------|-------------|
| Understand the "why" | `business-domain.md` | Problem, users, value proposition |
| Understand the "how" | `technical-domain.md` | Stack, architecture, integrations |
| Advanced patterns | `technical-patterns.md` | Realtime, WebSocket, RLS, build/debug |
| See the connection | `business-tech-bridge.md` | Business → technical mapping |
| Know the context | `decisions-log.md` | Why decisions were made |
| Current state | `living-notes.md` | Active issues and open questions |
| All of the above | Read all files in order | Full project intelligence |

## Usage

**New Team Member / Agent**:
1. Start with `navigation.md` (this file)
2. Read all files in order for complete understanding
3. Follow onboarding checklist in each file

**Quick Reference**:
- Business focus → `business-domain.md`
- Technical focus → `technical-domain.md`
- Decision context → `decisions-log.md`

## Integration

This folder is referenced from:
- `<CONFIG_ROOT>/.config/opencode/context/core/standards/project-intelligence.md` (standards and patterns)
- `<CONFIG_ROOT>/.config/opencode/context/core/system/context-guide.md` (context loading)

See `<CONFIG_ROOT>/.config/opencode/context/core/context-system.md` for the broader context architecture.

## Maintenance

Keep this folder current:
- Update when business direction changes
- Document decisions as they're made
- Review `living-notes.md` regularly
- Archive resolved items from decisions-log.md

**Management Guide**: See `<CONFIG_ROOT>/.config/opencode/context/core/standards/project-intelligence-management.md` for complete lifecycle management including:
- How to update, add, and remove files
- How to create new subfolders
- Version tracking and frontmatter standards
- Quality checklists and anti-patterns
- Governance and ownership

See `<CONFIG_ROOT>/.config/opencode/context/core/standards/project-intelligence.md` for the standard itself.
